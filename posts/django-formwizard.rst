=============================
Not Exactly Tim the Enchanter
=============================

:author: Kenneth
:category: django
:tags: django, forms, formwizard

Django has always been hailed as having great, awesome documentation. And, for the most part, this distinction has been deserved. But every once in a while, you find an area that's just...lacking. An area that you know exists but you've never gone into because it didn't come up and no one else used it but then you *finally* got a reason to use it...and then you find out it's so lacking in documentation that you have to dive into the source code.

This is one of those areas. Welcome to `Django Form Wizard`_.

What is Django Form Wizard?
===========================

At it's most basic, Django's Form Wizard (DFW from now on, OK?) is a way to auto-generate a set of views w/ a single form in each view. Most of the time it uses one template but you can have a different template for each view. Most wizards have numbered steps but you can have named ones if you want. Also, most wizards deal with standard forms (**highly** recommended) but can, of course, use formsets and model forms.

You've all used wizards before, I'm sure. A set of forms that you fill out in order and, at the end, something larger is assembled from the information you provided. Usually it's an account or the install of a program or something of the like. I needed to produce a new product in an online store.

You Got Your Chocolate In My Peanut Butter
==========================================

One thing I *hate* is putting logic where it doesn't belong. This is why you'll always see me importing my views classes (and creating view classes to begin with) into `urls.py`, even for views that go direct to template. As far as I'm concerned, `urls.py` is meant to be a mapping of regular expressions and nothing more.

DFWs don't let me have that separation of concerns, though (at least not in my experiences). I ended up having to import views, ``formset_factory`` and forms into ``urls.py`` **and** build out a couple of objects in the file. The messiness eats away at my CDO (OCD but in alphabetical order like it should be) even now, a week later. If this, alone, could be fixed, I'd be a much happier person. I should be able to provide a ``get_forms_list()`` method or ``forms_list`` class-level variable on the wizard view. Same goes for specifying the ``url_name``, but I'm getting ahead of myself.

The View, Part One
==================

For an area with very little documentation, there are quite a few ``WizardView`` sub-classes to pick from. The two most people will likely use are ``SessionWizardView`` or ``CookieWizardView``, both of which work the same way but store the user's progress in a different place. The former obviously stores it in their session, the latter in a cookie on their machine. There are also named variants of both, ``NamedUrlSessionWizardView`` and ``NamedUrlCookieWizardView``. I used the ``NamedUrlSessionWizardView`` because I wanted named steps in the URL (just looks nicer) and I wanted it stored in the user's session.

.. code-block:: python

    from django.contrib.formtools.wizard.views import NamedUrlSessionWizardView
    from django.core.files.storage import FileSystemStorage

    class ProductWizardView(LoginRequiredMixin, NamedUrlSessionWizardView):
        file_storage = FileSystemStorage()
        template_name = "seller/wizard_form.html"

Notice that I specify a file storage instance. If you don't specify this, and you need to support ``FileField`` or ``ImageField`` in your forms, you'll get errors from Django. This is something else I think could be handled by the views better. Seems to me that it should just use whatever the default/specified storage is for the rest of your project/application.

URLs
====

Now we jump to ``urls.py``.

.. code-block:: python
    
    from django.forms.formsets import formset_factory

    from app.forms import ProductModelForm, ShippingForm, PhotoForm
    from app.views import ProductWizardView

    shipping_formset = formset_factory(ShippingForm, max_num=25, extra=5)
    photo_formset = formset_factory(PhotoForm, max_num=25, extra=5)

    named_product_forms = (
        ("product", ProductModelForm),
        ("shipping", shipping_formset),
        ("photos", photo_formset)
    )

    product_wizard = ProductWizardView.as_view(named_product_forms,
        url_name="product_wizard_step")

    urlpatterns = patterns('',
        url(r"^productwizard/(?P<step>[-\w]+)/$", product_wizard,
            name="product_wizard_step"),
        url(r"^productwizard/$", product_wizard, name="product_wizard"),
    )

You can see what I mean about how messy ``urls.py`` has quickly become.

First, we import our forms and the view we stubbed out. Since I need multiple forms for building out shipping options and product photos, I spin up a couple of formset factories. For the record, I hate this implementation, too, if anyone wants to tackle a better invocation.

The tuple of two-tuples that is ``named_product_forms`` is where a bit of the magic happens. The first item in each tuple is the name of the step. This'll show up in your URL and you'll string-match this if you need to do special work on any given step (more on this in a minute). You pass this list of forms into your views ``as_view()`` method when you instantiate the view, along with a name for the **step** url version of your wizard view.

In your ``urlpatterns``, you'll define two URLs for the one view, one that has a variable for the step and one that doesn't. These can probably be combined but, in my experiments, DFWs aren't really friendly to you being too clever.

View, Part Two
==============

All DFWs have a method named ``done()`` that takes one explicit arg, ``form_list``, and then any kwargs you want to pass into it. This step is run when all of your forms have been submitted and they've all passed validation. Here is an approximation of my view's ``done()`` method.

.. code-block:: python

    def done(self, form_list, **kwargs):
        product_form = form_list[0]
        shipping_forms = form_list[1]
        image_forms = form_list[2]

        productext = self.create_product(product_form)
        shippings = self.create_shippings(productext, shipping_forms)
        images = self.create_images(productext, image_forms)

        if all([productext, shippings, images]):
            del self.request.session["wizard_product_wizard_view"]

            messages.success(self.request,
                _("Your product has been created."))
            return HttpResponseRedirect(self.get_success_url(productext))

        messages.error(self.request, _("Something went wrong creating your "
            "product. Please try again or contact support."))
        return HttpResponseRedirect(reverse("product_wizard"))

The first thing I do is assign my forms to different variables for ease of reach. I have a few methods on my class for creating each of my model types. Each of these methods returns either ``True`` or ``False``, which makes my call to ``any()`` the absolute easiest way to make sure they're all successful. If they are, I dump the wizard's variable from the session, set a message, and redirect (as you should always do after a ``POST``). If not, I set another message and redirect back to the wizard view. This'll send the user to the last step of the form, just in case something else has come up. If, somehow, the user got to the ``done()`` step before completing the entire wizard, they'd be redirected to the last step they completed.

The session variable is created by Django by appending, with underscores, an un-camel-cased version of your class name to the word "wizard". This isn't specified in the docs anywhere, but you can see the build up of it `here in Github`_. I found it just by examining the ``request.session`` object in a PDB shell.

One other method I overrode on the view is ``get_form_kwargs``.

.. code-block:: python

    def get_form_kwargs(self, step):
        if step == "product":
            return {"user": self.request.user}
        return {}

Each step calls this method with either its index value or its name, depending on the type of DFW you're using. As you can see, I check to see if it's the product step and, if so, I return a dict with a ``user`` variable set to the current request's user.

Forms And The Template
======================

I haven't shown any forms because they're not really special. I recommend you stick with non-``ModelForm`` forms, though. Using ``ModelForm`` forms seems like a great idea until you remember that no forms are really processed, other than making sure they pass ``is_valid()``, until the ``done()`` step. That means that if you have a `ModelForm` on steps one and two and the form on step two relies on the model instance created by the form in step one, step two's form will never be valid.

The template, also, isn't really special, in and of itself. In my implementation, though, I used `django-crispy-forms`_ and that presented a small problem to my normal flow.

Usually, in templates, I do something like the following to render a form:

.. code-block:: html

    {% load crispy_forms_tags %}
    {% crispy form %}

That'll work great with DFWs with one small change in your form's helper.

.. code-block:: python

    class WizardForm(forms.Form):
        [...]

        def __init__(self, *args, **kwargs):
            self.helper = FormHelper()
            self.helper.form_tag = False
            [...]

I had to tell my forms not to render the form tag since I needed to be able to override the ``enctype`` on the tag. I also left off any submit buttons since you can add "Previous" and "First" buttons to the forms too.

Conclusion
==========

Hopefully this gives you a pretty good idea of how to implement DFWs in your own product. They're a fairly useful way to create new items or lead a user through a lengthy form or process. Sadly it's not really useful for editing since it's difficult to pass in instances in the appropriate places. I'd love to see the docs expanded on this, both with actual documentation and with better examples.


.. _Django Form Wizard: https://docs.djangoproject.com/en/1.4/ref/contrib/formtools/form-wizard/
.. _django-crispy-forms: http://django-crispy-forms.readthedocs.org/en/d-0/
.. _here in Github: https://github.com/django/django/blob/master/django/contrib/formtools/wizard/storage/base.py#L16


