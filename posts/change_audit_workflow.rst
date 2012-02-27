=======================
Change Request Workflow
=======================

:author: Kenneth
:date: 2012-02-26 11:01
:status: draft
:category: django
:tags: django, workflow, python

Recently, at work, we had need of a workflow where users of a certain access level could submit changes to existing records, but the changes had to be approved by a higher-leveled user before they went live. That 
editor-level user could either approve or deny the changes. We didn't make it to where they could approve *part* of a change but not all of it, but it wouldn't be hard to extend our workflow to add that in. We thought 
you might be interested in it, so we wanted to show what we did.

Models
======

First, let's start with the models.

.. code-block:: django

    class POIAbstract(LumberjackModel, models.Model):
        category = models.ForeignKey(Category, related_name="%(class)s_points")
        name = models.CharField(max_length=255)
        address = models.CharField(max_length=255)
        address2 = models.CharField(max_length=255, blank=True)
        city = models.CharField(max_length=100)
        state = USPostalCodeField()
        zip_code = models.CharField(max_length=10)
        phone = PhoneNumberField(blank=True, default="")
        url = models.URLField(blank=True, default="")
        point = models.PointField(blank=True, null=True, editable=False)
        objects = models.GeoManager()

        class Meta:
            abstract = True

        def __unicode__(self):
            return self.name

        @property
        def coords(self):
            """
            Return tuple of lat,lng
            """
            if self.point:
                return (self.point.get_coords()[1], self.point.get_coords()[0])
            return (None, None)

        @property
        def full_address(self):
            """
            Return a string of the full address
            """
            addresses = [self.address, self.address2, self.city, self.state,
                self.zip_code, "USA"]
            return ", ".join(filter(lambda x: len(x) > 0, addresses))


    class POI(POIAbstract):
        """
        Points of Interest model.
        """
        pass


    class POIChange(POIAbstract):
        """
        Holds proposed changes to POIs
        """
        STATUS_CHOICES = (
            (0, "Pending"),
            (1, "Denied"),
            (2, "Approved")
        )

        poi = models.ForeignKey(POI, related_name="changes")
        user = models.ForeignKey(User, related_name="poi_changes")
        submitted_on = models.DateField(auto_now_add=True, editable=False)
        approved_by = models.ForeignKey(User, related_name="poi_approvals",
            blank=True, null=True, editable=False)
        approved_on = models.DateField(blank=True, null=True, editable=False)
        status = models.PositiveSmallIntegerField(choices=STATUS_CHOICES,
            default=0, editable=False)

        class Meta:
            ordering = ["status", "-submitted_on"]

As you can see, there's not really anything too interesting about the models. We have an abstract model that we inherit both of our other models from. The approved record model is just the abstract model without it's 
``abstract = True`` setting. The change model, though, adds a few fields.

First we point to the record we're changing. Then we hold on to the user that submitted the changes, and the time of the request. We also want to have a record of who approved/denied it and when. And, of course, we 
need to know what the status of the change is. That'll let us change our minds later on.

Forms
=====

We usually end up building forms after we build models (more on this when we finish GSWD_), so let's look at them next.

.. code-block:: django

    class POIForm(forms.ModelForm):
        latitude = forms.FloatField(required=False,
            widget=forms.HiddenInput())
        longitude = forms.FloatField(required=False,
            widget=forms.HiddenInput())

        class Meta:
            model = POI


    class POIChangeForm(forms.ModelForm):
        latitude = forms.FloatField(required=False,
            widget=forms.HiddenInput())
        longitude = forms.FloatField(required=False,
            widget=forms.HiddenInput())

        class Meta:
            model = POIChange
            widgets = {
                "poi": forms.HiddenInput(),
                "user": forms.HiddenInput()
            }

I've left out some of the boilerplate and ``Layout`` bits from django-uni-form_ (we haven't upgraded to django-crispy-forms_ yet) but you get the general idea. Honestly, we could have made the second form inherit from 
the first and saved a bit of typing/space, but I guess we missed that. Both forms, ultimately, show the same thing. The latter form, though, holds onto a few extra fields that we need and that we'll set in the view.

Speaking of views, let's check them out.

Views
=====

We're not going to look at the view that creates the original POI. It's just a standard 
``CreateView`` that specifies our ``POIForm`` as the ``form_class``. We have a couple of handy 
mixins on the views that let us control permissions and redirects, but we'll talk about them in 
another blog post.

The view we *do* want to look at is our ``POIUpdateView`` which is the one that let's users submit 
changes for a particular POI. Now, this view is the one that's linked to for each record on the 
list page; we never link to a view where a user can directly update a POI, not even for 
editors/superusers. So, here's our ``POIUpdateView``:

.. code-block:: django

    # pydanny note: Careful using non-core Mixins like SetHeadlineMixin without an explanation
    #     You'll confuse a lot of people like me. ;)
    class POIUpdateView(LoginRequiredMixin, PermissionRequiredMixin,
        SuccessURLRedirectListMixin, SetHeadlineMixin, CreateView):
        """
        View allows users to propose changes to current POIs.
        """

        form_class = POIChangeForm
        headline = "Edit point of interest"
        model = POIChange
        permission_required = "points.change_poi"
        success_list_url = "cms_points_list"
        template_name = "cms/points/poi_form_edit.html"

        def get_initial(self):
            """
            Do you believe in magic, in a young devs heart?
            How the code can free 'em whenever it starts,
            and it's magic, if the code is groovy.

            Use POI information for initial data in POIChangeForm.
            """
            poi = POI.objects.get(pk=self.kwargs["pk"])
            initial = poi.__dict__.copy()
            del initial["_state"]
            initial.update({
                "category": poi.category,
                "latitude": poi.point.get_coords()[1],
                "longitude": poi.point.get_coords()[0],
                "user": self.request.user,
                "poi": poi
            })
            return initial

        def post(self, request, pk, *args, **kwargs):
            response = super(POIUpdateView, self).post(request, pk, *args, **kwargs)

            url = settings.CMS_URL + reverse("cms_points_change_detail",
                kwargs={"pk": self.object.pk})
            message = render_to_string("cms/points/email/admin_email.html", {"user":
                self.object.user.get_full_name(), "url": url})
            mail_admins("POI Change Request", message)

            return response

I think how this view works is pretty cool. It's a fairly standard ``CreateView`` that points to 
our ``POIChange`` model. We don't just start with a blank ``POIChange``, though. By overriding 
``get_initial`` to load the ``POI`` with the ``PK`` that comes through in the URL, we can set the 
beginning data of the record. We fetch the instance, update our initial data with its values, and 
then pass it on through to the form.

Once the form is valid, a method I don't show above, called ``form_valid``, is fired by Django as part of its form-based generic view workflow and then we log the change in our logger, send a message to the user 
through Django's ``messages`` app, and then our ``post`` method gets called.  Learning the workflow order of ``CreateView`` and ``UpdateView`` (and, ultimately, ``FormView``) will save you a huge amount of time when 
you start customizing these things.  In our ``post`` method, we render out an email to the admins and then return our response, which, thanks to our ``SuccessURLRedirectListMixin`` will redirect the user to the route 
named in ``success_list_url``.

Now, all we've really done is create a new record. It still has to be approved. We do that in our 
next view, ``POIChangeApprovalView``, which the editor/superuser gets to through another list 
view. They can also reach it by clicking the link provided to them in the email.

.. code-block:: django

    class POIChangeApprovalView(LoginRequiredMixin, SuperuserRequiredMixin,
        DetailView):

        model = POIChange
        template_name = "cms/points/poi_change_detail.html"

        def post(self, request, pk):
            approval = request.POST.get("approval", None)
            if approval:
                if approval == "approve":
                    self._approved()
                else:
                    self._denied()
                return HttpResponseRedirect(reverse("cms_points_change_list"))

            return HttpResponseForbidden()

        def _approved(self):
            """
            It's approved!
            """
            poi = self.get_object()
            data = poi.__dict__.copy()
            del data["_state"]
            data.update({
                "category": poi.category.pk,
                "latitude": poi.coords[0],
                "longitude": poi.coords[1],
            })
            form = POIForm(data, instance=poi.poi)
            if form.is_valid():
                form.save()

                poi.status = 2
                poi.approved_by = self.request.user
                poi.approved_on = date.today()
                poi.save()

                messages.success(self.request, "Point of interest updated.")

                if poi.user.email:
                    message = render_to_string("cms/points/email/approved.html",
                        {"poi_name": poi.name})
                    send_mail("OUR CLIENT - Change Request Approved",
                        message, settings.EMAIL_HOST_USER, [poi.user.email])

        def _denied(self):
            """
            No way Jose
            """
            poi = self.get_object()
            poi.status = 1
            poi.approved_by = self.request.user
            poi.approved_on = date.today()
            poi.save()

            messages.success(self.request,
                "Point of interest '%s' has not been updated." % poi.poi.name)

            if poi.user.email:
                message = render_to_string("cms/points/email/denied.html",
                    {"poi_name": poi.name})
                send_mail("OUR CLIENT - Change Request Denied",
                    message, settings.EMAIL_HOST_USER, [poi.user.email])

This view is really straightfoward. The editor clicks one of two buttons, both of which point to 
this view. One contains a ``POST`` variable indicating approval, the other indicating that the 
change has been denied. Then, based on the value, we peform the same action on the ``POIChange``.

If the change was denied, we just set the status on the change to our denied flag, set the date 
and user, and then save it.

If it was approved, we create an instance of the ``POIForm`` with the changed ``POI`` as the 
edited instance and our ``POIChange``'s ``__dict__`` as the new data. Since they're copies of each 
other, aside from the changes in the change model, of course, only the changed data really gets 
updated. We make sure the form is still valid (some GeoDjango_ stuff I left out of the form above) 
and then save the updated instance. We also update the ``POIChange`` so it holds the new status, 
the approving user and date.

Regardless of the action taken, we send off an email to the user that submitted the change, 
letting him or her know what happened.

Summary
=======

This has, so far, been a great workflow for our users. They're able to trust that the data going 
out is verified and safe, but if anything gets out of date, we can change it ourselves or let the community of users tell us about the new data.

There is a lot of stuff I didn't cover, what the ``Point`` field holds on to, how to actually use GeoDjango, what each of our custom mixins does (we're planning on releasing these as a package soon), and lots of other 
stuff. If you have questions/comments, hit us up on Twitter_. Also, thanks to `Daniel Greenfeld`_ for a couple of edits.

.. _GSWD: http://gettingstartedwithdjango.com
.. _django-uni-form: https://github.com/pydanny/django-uni-form
.. _django-crispy-forms: https://github.com/maraujop/django-crispy-forms
.. _GeoDjango: http://geodjango.org
.. _Twitter: http://twitter.com/brack3t
.. _Daniel Greenfeld: http://pydanny.github.com
