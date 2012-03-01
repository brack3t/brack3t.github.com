=================
Our Custom Mixins
=================

:author: Chris
:category: django
:tags: django, CBVs
:status: draft

Let's just start out and say it, ``Class Based Views``. Ooohhhh. Unfortunately the topic of class based views is 
thought of as somewhat of a dark art in the Django community. It doesn't help that the documentation is still very 
lacking but I find a lot of people, especially on Reddit, that refuse to use them. For whatever reason, it's a hard 
pill for some to swallow.

Before Djangocon 2011, we started playing with class based views. At first they seem like a nightmare and without 
decent docs, you can get frustrated really fast. Skip forward to today and I can't imagine writing old function based 
views again. Some argue that the ``generic views`` are only for generic applications and that somehow, their work is far too 
custom and complex to be handled in a generic class based view. Based on my experience, they would be mostly wrong. I'll 
leave that mostly in there for the 1-2% that that might apply to.

We plan on covering generic class based views extensively with GSWD_. Today, I'd like to share some mixins we 
have cooked up on a rather large client project that have helped us out tremendiously.

For those of you not familiar with decorating class based views, check out the Django documentation on 
`decorating the class`_.


LoginRequiredMixin
==================

.. code-block:: django

    class LoginRequiredMixin(object):
        """
        View mixin which verifies that the user has authenticated.

        NOTE:
            This should be the left-most mixin of a view.
        """

        @method_decorator(login_required)
        def dispatch(self, *args, **kwargs):
            return super(LoginRequiredMixin, self).dispatch(*args, **kwargs)


This mixin is rather simple and is generally the first inherited class in any of our views. If we don't have an authenticated user 
there's no need to go any further. If you've used Django before you are probably familiar with the ``login_required`` decorator. 
All we are doing here is requiring a user to be authenticated to be able to get to this view.

While this doesn't look like much, it frees us up from having to manually overload the dispatch method on every single view that 
requires a user to be authenticated. If that's all that is needed on this view, we just saved 3 lines of code. Example usage below.

.. code-block:: django

    from django.views.generic import TemplateView

    from myapp.mixins import LoginRequiredMixin


    class SomeSecretView(LoginRequiredMixin, TemplateView):
        template_name = "path/to/template.html"

        def get(self, request):
            return self.render_to_response({})


PermissionRequiredMixin
=======================

.. code-block:: django

    class PermissionRequiredMixin(object):
        """
        View mixin which verifies that the logged in user has the specified
        permission.

        Class Settings
        `permission_required` - the permission to check for.
        `login_url` - the login url of site
        `redirect_field_name` - defaults to "next"
        `raise_exception` - defaults to False - raise 403 if set to True

        Example Usage

            class SomeView(PermissionRequiredMixin, ListView):
                ...
                # required
                permission_required = "app.permission"

                # optional
                login_url = "/signup/"
                redirect_field_name = "hollaback"
                raise_exception = True
                ...
        """
        login_url = settings.LOGIN_URL
        permission_required = None
        raise_exception = False
        redirect_field_name = REDIRECT_FIELD_NAME

        def dispatch(self, request, *args, **kwargs):
            original_return_value = super(PermissionRequiredMixin, self).dispatch(
                request, *args, **kwargs)

            # Verify class settings
            if self.permission_required == None or len(
                self.permission_required.split(".")) != 2:
                raise ImproperlyConfigured("'PermissionRequiredMixin' requires "
                    "'permission_required' attribute to be set.")

            has_permission = request.user.has_perm(self.permission_required)

            if not has_permission:
                if self.raise_exception:
                    return HttpResponseForbidden()
                else:
                    path = urlquote(request.get_full_path())
                    tup = self.login_url, self.redirect_field_name, path
                    return HttpResponseRedirect("%s?%s=%s" % tup)

            return original_return_value

This mixin was actually written, I believe, by `Daniel Sokolowski`_ (`code here`_). 

The permission required mixin has been very handy for our client's custom CMS. Again, rather than overloading the 
dispatch method manually on every view that needs to check for the existence of a permission, we inherit this class 
and set the ``permission_required`` class attribute on our view. If you don't specify ``permission_required`` on 
your view, an ``ImproperlyConfigured`` exception is raised reminding you that you haven't set it.

The one limitation of this mixin is that it can **only** accept a single permission. It would need to be modified to 
handle more than one. We haven't needed that yet, so this has worked out well for us.

In our normal use case for this mixin, ``LoginRequiredMixin`` comes first, then the ``PermissionRequiredMixin``. If we 
don't have an authenticated user, there is no sense in checking for any permissions.

    .. role:: info-label
        :class: "label label-info"

    :info-label:`note` If you are using Django's built in auth system, ``super users`` automatically have all permissions in your system.

SuperuserRequiredMixin
======================

.. code-block:: django

    class SuperuserRequiredMixin(object):
        login_url = settings.LOGIN_URL
        raise_exception = False
        redirect_field_name = REDIRECT_FIELD_NAME

        def dispatch(self, request, *args, **kwargs):
            original_return_value = super(
                SuperuserRequiredMixin, self).dispatch(
                request, *args, **kwargs)

            if not request.user.is_superuser:
                if self.raise_exception:
                    return HttpResponseForbidden()
                else:
                    path = urlquote(request.get_full_path())
                    tup = self.login_url, self.redirect_field_name, path
                    return HttpResponseRedirect("%s?%s=%s" % tup)

            return original_return_value

Another permission based mixin. This is specifically for requiring a user to be a super user. Comes in handy for tools that only privilaged 
users should have access to.


UserFormKwargsMixin
===================

.. code-block:: django

    class UserFormKwargsMixin(object):
        """
        CBV mixin which puts the user from the request into the form kwargs.
        Note: Using this mixin requires you to pop the `user` kwarg
        out of the dict in the super of your form's `__init__`.
        """
        def get_form_kwargs(self, **kwargs):
            kwargs = super(UserFormKwargsMixin, self).get_form_kwargs(**kwargs)
            kwargs.update({"user": self.request.user})
            return kwargs

In our clients CMS, we have a lot of form based views that require a user to be passed in for permission based form tools. For example, 
only super users can delete or disable certain objects. To custom tailor the form for users, we have to pass that user instance into the form 
and based on their permission level, change certain fields or add specific options within the forms ``__init__`` method.

This mixin automates the process of overloading the ``get_form_kwargs`` method and stuffs the user instance into the form kwargs. We can then 
pop the user off in the form and do with it what we need. **Always** remember to pop the user from the kwargs before calling ``super`` on your 
form, otherwise the form gets an unexpected kwarg and everything blows up. Example usage: 

.. code-block:: django

    from django.views.generic import CreateView

    from myapp.mixins import LoginRequiredMixin, UserFormKwargsMixin


    class SomeForm(forms.ModelForm):
        class Meta:
            model = SomeModel

        def __init__(self, *args, **kwargs):
            self.user = kwargs.pop("user", None)
            super(SomeForm, self).__init__(*args, **kwargs)

            if self.user.is_superuser:
                ...
                Allow them to do something awesome.
                ...


    class SomeSecretView(LoginRequiredMixin, UserFormKwargsMixin,
        TemplateView):

        form_class = SomeForm
        model = SomeModel
        template_name = "path/to/template.html"


UserKwargModelFormMixin
=======================

``Show usage of the new form mixin``


SuccessURLRedirectListMixin
===========================

.. code-block:: django

    class SuccessURLRedirectListMixin(object):
        """
        Simple CBV mixin which sets the success url to the list view of
        a given app. Set success_list_url as a class attribute of your
        CBV and don't worry about overloading the get_success_url.

        This is only to be used for redirecting to a list page. If you need
        to reverse the url with kwargs, this is not the mixin to use.
        """
        success_list_url = None

        def get_success_url(self):
            return reverse(self.success_list_url)


SetHeadlineMixin
================

.. code-block:: django

    class SetHeadlineMixin(object):
        """
        Mixin allows you to set a static headline through a static property on the
        class or programmatically by overloading the get_headline method.
        """
        headline = None

        def get_context_data(self, **kwargs):
            kwargs = super(SetHeadlineMixin, self).get_context_data(**kwargs)
            kwargs.update({"headline": self.get_headline()})
            return kwargs

        def get_headline(self):
            if self.headline is None:
                raise ImproperlyConfigured(u"%(cls)s is missing a headline. Define "
                    u"%(cls)s.headline, or override "
                    u"%(cls)s.get_headline()." % {"cls": self.__class__.__name__
                })
            return self.headline


.. _GSWD: http://gettingstartedwithdjango.com
.. _decorating the class: https://docs.djangoproject.com/en/dev/topics/class-based-views/#decorating-the-class
.. _Daniel Sokolowski: https://github.com/danols
.. _code here: https://github.com/lukaszb/django-guardian/issues/48
