=====================
Ajax and Django Views
=====================

:author: Chris
:category: django
:tags: django, ajax, jquery
:status: draft


Django automatic CSRF
=====================

https://docs.djangoproject.com/en/dev/ref/contrib/csrf/#ajax


Ajax & Form Field Errors
========================

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


Ajax Views
==========

.. code-block:: django

    class NavigationItemAjaxSortView(LoginRequiredMixin, PermissionRequiredMixin,
        View):

        permission_required = "navigations.change_navigation"

        def post(self, request):
            pk = request.POST.get("pk", None)

            if pk:
                nav_item = get_object_or_404(NavigationItem, pk=pk)
                form = NavigationItemForm(request.POST, instance=nav_item)
                if form.is_valid():
                    form.save()
                    return HttpResponse(json.dumps("success"),
                        mimetype="application/json")
                return HttpResponseForbidden(json.dumps(form.errors),
                    mimetype="application/json")
            raise Http404
