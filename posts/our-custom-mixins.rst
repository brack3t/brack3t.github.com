=================
Our Custom Mixins
=================

:author: Chris
:tags: django, CBVs
:status: draft

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
