=====================
Ajax and Django Views
=====================

:author: Chris and Kenneth
:category: django
:tags: django, ajax, jquery
:status: draft

It seems that cleanly and easily doing AJAX views in Django is an area that gives a lot of people trouble. We like to do views as straight HTTP if at all possible, but there are always interactions that would be better served by *not* having a page load. We're also big fans of `django-tastypie`_ but it's a whole other ball of wax on its own, especially if you want to have views that write to the database. So, for the purposes of getting everyone up to speed doing AJAX with Django, we'll ignore Tastypie for now and just stick with ordinary views.


Django automatic CSRF
=====================

To start things off, put `this bit of Javascript`_ from the Django docs into a script that's loaded on all the pages where you'll be needing to perform AJAX views. This allows you to ignore the CSRF token for AJAX views. While this sounds more dangerous, the same-origin policy of AJAX requests should provide a decent amount of protection already. If you're really concerned, don't use this script and fetch a new CSRF token every time you submit a form, basically rebuilding the HTML form after every submission.

AJAX & Form Field Errors
========================

Before we get to the Django side, there are a few small scripts that we recycle on every project.

.. code-block:: javascript

    function apply_form_field_error(fieldname, error) {
        var input = $("#id_" + fieldname),
            container = $("#div_id_" + fieldname),
            error_msg = $("<span />").addClass("help-inline ajax-error").text(error[0]);

        container.addClass("error");
        error_msg.insertAfter(input);
    }

    function clear_form_field_errors(form) {
        $(".ajax-error", $(form)).remove();
        $(".error", $(form)).removeClass("error");
    }

These two functions are pretty self-explanatory, but I'll go over them anyway. The first, ``apply_form_field_error``, takes a field name and an error, finds the field on the page and sets the error text to the passed-in error. Our selectors here take advantage of how `Twitter Bootstrap`_ renders forms, so you may need to change the selectors to match your own markup. The second, ``clear_form_field_errors``, removes the above text and classes, basicaly resetting the form. It's important to run this script before every AJAX submission so you don't get doubled-up errors if someone submits invalid data on the same field twice. You could add it to jQuery's ``beforeSend`` if you don't want to have to think about it.

There are a couple more useful utility functions that we thought might be useful for some people.

.. code-block:: javascript

    function django_message(msg, level) {
        var levels = {
            warning: 'alert',
            error: 'error',
            success: 'success',
            info: 'info'
        },
        source = $('#message_template').html(),
        template = Handlebars.compile(source),
        context = {
            'tags': levels[level],
            'message': msg
        },
        html = template(context);

        $("#message_area").append(html);
    }

    function django_block_message(msg, level) {
        var source = $("#message_block_template").html(),
            template = Handlebars.compile(source),
            context = {level: level, body: msg},
            html = template(context);

        $("#message_area").append(html);
    }

These two scripts mirror Django's ``messages`` app's functionality. Both use Handlebars_ to render the template, but you could use any Javascript templating library you'd like. Our templates look like so:

.. code-block:: html

    {% load verbatim_tag %}
    <script id="message_template" type="text/x-handlebars-template">
        {% verbatim %}
        <div class="alert alert-{{tags}} fade in" data-alert="{{tags}}">
            <a class="close" title="Close" href="#" data-dismiss="alert">&times;</a>
            {{{message}}}
        </div>
        {% endverbatim%}
    </script>

    <script id="message_block_template" type="text/x-handlebars-template">
        {% verbatim %}
        <div class="alert alert-block alert-{{level}} fade in">
            <a class="close" title="Close" href="#" data-dismiss="alert">&times;</a>
            {{{body}}}
        </div>
        {% endverbatim %}
    </script>

The ``verbatim`` tag_ there is from Eric Florenzano and makes including Javascript templates in your Django-parsed HTML really easy. We include these in a base template and provide a spot in the rest of the document to attach them to. Again, this is based largely on Twitter Bootstrap, so your markup will vary.

AJAX Views
==========

So now let's get down to the good stuff. The following view is very generic and only shows the basic concept, but we're sure you'll get the gist of it.

.. code-block:: django

    from django.http import HttpResponse, HttpResponseBadRequest
    from django.utils import simplejson as json
    from django.views.generic import UpdateView

    from braces.views import LoginRequiredMixin, PermissionRequiredMixin

    class PonyAjaxUpdateView(LoginRequiredMixin, PermissionRequiredMixin, UpdateView):

        form_class = PonyForm
        model = Pony
        permission_required = "ponies.change_pony"

        def form_valid(self, form):
            """
            If the request is ajax, save the form and return a json response.
            Otherwise return super as expected.
            """
            if self.request.is_ajax():
                self.object = form.save()
                return HttpResponse(json.dumps("success"),
                    mimetype="application/json")
            return super(PonyAjaxUpdateView, self).form_valid(form)

        def form_invalid(self, form):
            """
            We haz errors in the form. If ajax, return them as json.
            Otherwise, proceed as normal.
            """
            if self.request.is_ajax():
                return HttpResponseBadRequest(json.dumps(form.errors),
                    mimetype="application/json")
            return super(PonyAjaxUpdateView, self).form_invalid(form)

Again, nothing special in the view. We use an ``UpdateView`` so we can, technically, still use the view without AJAX. Assuming that the POST data that comes in validates on the form, our ``form_valid`` method will fire, which checks to see if the request was made via AJAX and, if so, returns a success string. Quite often we like to return a serialized version of the object that was just created or updated, but that takes some special considerations when it comes to Django model objects. If you don't need the object back, returning a standard ``HttpResponse`` or one with a message, like demonstrated above, is enough. If your view creates new objects or deletes old ones, returning proper status codes, like ``201`` for ``Created`` is a very polite thing to do, especially if you think your view will end up as part of an ad hoc API.

Similarly, above, if the form is invalid, we serialize the form errors (note: not the ``non_field_errors()`` errors) and send them back to the view. The script we wrote above, ``apply_form_field_error`` can be called in a loop for each error in the list and update your form so the users know what they did wrong.

    .. role:: info-label
        :class: "label label-info"

    :info-label:`note` Did you notice the *braces* package we used in the above view? That's a package we released from a previous blog post on `custom class-based view mixins`_. You can get it on Github_ or PyPI_.

Form Errors
-----------

The difference between ``form.errors`` and ``form.non_field_errors()``: ``form.errors`` are any errors directly related to a field in your form, ``form.non_field_errors()`` would include errors raised by a custom ``clean()`` method and are put into a special "field" called ``__all__``.


The jQuery Side
===============

HTML
----

.. code-block:: html

    <form id="pony_form" method="POST" action="{% url pony_update_view pony.pk %}">
        {% csrf_token %}
        <label for="id_name">Pony Name</label>
        <input type="text" name="name" id="id_name">
        <input type="submit" value="Submit">
    </form>

jQuery
------

.. code-block:: javascript

    $(document).on("submit", "#pony_form", function(e) {
        e.preventDefault();
        var self = $(this),
            url = self.attr("action"),
            ajax_req = $.ajax({
                url: url,
                type: "POST",
                data: {
                    name: self.find("#id_name").val()
                },
                success: function(data, textStatus, jqXHR) {
                    django_message("Pony saved successfully.", "success");
                },
                error: functior(data, textStatus, jqXHR) {
                    var errors = $.parseJSON(data.responseText);
                    $.each(errors, function(index, value) {
                        if (index === "__all__") {
                            django_message(value[0], "error");
                        } else {
                            apply_form_field_error(index, value);
                        }
                    });
                }
            });
    });


.. _this bit of Javascript: https://docs.djangoproject.com/en/dev/ref/contrib/csrf/#ajax
.. _Twitter Bootstrap: http://twitter.github.com/bootstrap
.. _Handlebars: http://handlebarsjs.com
.. _tag: https://gist.github.com/629508
.. _django-tastypie: http://tastypieapi.org
.. _Github: https://github.com/brack3t/django-braces
.. _PyPI: http://pypi.python.org/pypi/django-braces/
.. _custom class-based view mixins: http://brack3t.com/our-custom-mixins.html
