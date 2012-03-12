===========================
User-friendlier model forms
===========================

:author: Kenneth
:category: django
:tags: django, python, forms, models
:status: draft

Recently, in our large client project, we had need of fields, in a model form, that accepted multiple types of input, but sanitized the data for the model. For example, the ``rent`` field, on the form, needs to handle a rent range (e.g. 900-1200), a single amount, or be overridden or extended by other bits of information, like "call for details" or "on approved credit". Obviously we don't want to have to parse this out every time we read the data. So, enter our fields that tear data apart and put it together every time it passes through.

Model
=====

Let's go over our ``Rent`` model first. It's an abstract model so we can use it in multiple places (we have more than one logical model in the system that needs to deal with rent, this way we can use it multiple places without having to hold on to a huge amount of joins). We have several other abstract models that perform the same actions as our ``Rent`` model, but I won't show them here.

.. code-block:: django

    from django.core.exceptions import ValidationError
    from django.db import models


    class Rent(models.Model):
        rent_low = models.PositiveIntegerField()
        rent_high = models.PositiveIntegerField(blank=True, null=True)
        rent_percent_income = models.FloatField(blank=True, null=True)
        rent_oac = models.BooleanField(default=False)
        rent_call_for_details = models.BooleanField(default=False)
        rent_up_to = models.PositiveIntegerField(blank=True, null=True)

        class Meta:
            abstract = True

        def clean(self):
            super(Rent, self).clean()
            if self.rent_high and self.rent_high <= self.rent_low:
                raise ValidationError("Invalid rent range.")

            if self.rent_percent_income or self.rent_call_for_details or \
               self.rent_up_to:

                self.rent_low = 0
                self.rent_high = None

        @property
        def rent(self):
            if self.rent_call_for_details:
                return u"Call for details."

            if self.rent_up_to:
                return u"Up to $%d" % self.rent_up_to

            response = ""

            if self.rent_percent_income:
                response += u"%g%% of income." % self.rent_percent_income

            if self.rent_high:
                response +=  u"%d-%d" % (self.rent_low, self.rent_high)

            if self.rent_low > 0 and not self.rent_high:
                response += u"%d" % self.rent_low

            if self.rent_oac:
                response += " On approved credit."

            return response

The one "gotcha" here, that you may not get right away, is the ``super(Rent, self).clean()`` at the top of the ``clean()``. We explicitly call it here to make sure the cleaning continues up the chain in our models that extend ``Rent`` and the other extended models (as mentioned, we have several models created and use this way). You'll notice in the model we have a field for each of our states, the low and high values of rent, and the other fields that override the rent output value. We also have a class property of ``rent`` that we can call on the extending models to get the computed rent value.

That property doesn't do anything really interesting except return a value based on the field values. The clean is a little more interesting for how it sets ``rent_low`` to 0 and empties out ``rent_high`` when their values no longer matter.

Form
====

.. code-block:: django
    import re

    from django.core.validators import RegexValidator

    [...]

    integer_range = RegexValidator(
        regex=re.compile(r"^[0-9]*(-[0-9]*)?$"),
        message="Please enter a valid number, or a range in the format: 100-200",
        code="invalid"
    )


    class FloorplanBaseForm(CommunityKwargModelFormMixin, UserKwargModelFormMixin,
        forms.ModelForm):

        rent = forms.CharField(max_length=75, required=False,
            validators=[integer_range])

        class Meta:
            model = Floorplan


        def __init__(self, *args, **kwargs):
            super(FloorplanBaseForm, self).__init__(*args, **kwargs)

            [...]

            if self.instance.pk:
                set_custom_fields(self, ["rent", "deposit", "promo", "sq_ft"])

        def clean(self):
            super(FloorplanBaseForm, self).clean()
            data = self.cleaned_data

            [...]

            if data.get("rent", None) and not data["rent_call_for_details"] and not\
                data["rent_percent_income"] and not data["rent_up_to"]:

                split_ranges(self, "rent")

            clean_custom_fields(self, data, ["rent", "rent_call_for_details",
                "rent_up_to", "rent_percent_income"],
                "You must enter a value for rent.", "rent")

            return data

I've removed bits of the form that deal with other fields like ``rent`` since I'm not showing anything about them. This is, more or less, an abstract form. We never render it, but we extend it to support our specific floorplan types. In those extending forms, we tell ``rent_low`` and ``rent_high`` to be excluded. In this form, though, we provide a single ``rent`` field that has a regular expression validator on it to ensure that it contains an interger or two integers separated by a hyphen. This lets the users enter data as more-or-less natural text instead of having to tab through a bunch of fields or enter the data in a weird format.

You'll notice three custom methods being called, ``set_custom_fields``, ``split_ranges``, and ``clean_custom_fields``. We'll cover them next.

Custom methods
==============

Let's go over these one at a time.

.. code-block:: django

    def clean_custom_fields(form, cleaned_data, fields, error_msg, field):
        """
        Make sure at least one required option has been supplied.
        """
        if not any([cleaned_data.get(f, None) for f in fields]):
            form.errors[field] = form.error_class([error_msg])

Since we have more than one field to clean, but they can be used in several different combinations, we have to make sure that at least one of the fields is provided. The ``any`` method from the Python standard library is amazing useful for this. We pass in the form, because, again, we use this multiple places, our form's cleaned data, the fields we want checked, an error message, and the field to highlight if none of them are provided. This is a fairly useful and flexible solution that has, so far, fulfilled all of our needs.

Next is the ``split_ranges`` field.

.. code-block:: django

    def split_ranges(form, field):
        """
        Split custom range fields into model fields.
        """
        try:
            low, high = form.cleaned_data[field].split("-")
            form.instance.__setattr__(field + "_low", int(low))
            form.instance.__setattr__(field + "_high", int(high))
        except ValueError:
            form.instance.__setattr__(field + "_low",
                int(form.cleaned_data[field]))
            form.instance.__setattr__(field + "_high", None)

This small little method takes our unified field in the form and splits it out into the ``high`` and ``low`` fields on the model. We feel like doing ``__setattr__`` is a little dirty, but it solves the problem without us having to pass in a huge number of fields. Since our fields are named reliably and similarly, we're able to set fields without knowing all the names.

Also, notice how we use the ``ValueError`` that'll be throw by not having a ``high`` value to set on the form to trigger it being set to ``None``, exactly what our model is expecting already.

.. code-block:: django

    def set_custom_fields(form, fields):
        """
        Combine low/high fields into the range fields.
        """
        for field in fields:
            if form.instance.__getattribute__(field + "_high"):
                form.fields[field].initial = u"%d-%d" % (
                    form.instance.__getattribute__(field + "_low"),
                    form.instance.__getattribute__(field + "_high"))

            if form.instance.__getattribute__(field + "_low") > 0 and not \
                form.instance.__getattribute__(field + "_high"):

                form.fields[field].initial = form.instance.__getattribute__(
                    field + "_low")

This method is the reverse of the one above. We look at the initial data that is passed in when editing a model instance and combine our values so they match what the user would have already entered. Again, ``__getattribute__`` feels a little dirty, using a marked-as-private method and all, but it solves the problem at hand. I suppose we could have created our own form class, adding in ``setattribute`` and ``getattribute`` methods that just call these on their own, but that didn't seem amazingly necessary.

So, that model and that form combined with those methods lets us handle natural language entries for somewhat complex data. Granted, our use case would be negated by adding an extra field, but it's less friendly. One of our biggest goals on any client work we do is to make it user-friendly and a solid user experience all the way around. This bit of extra work has helped us do that quickly and easily.

Hopefully this gives you some ideas on how to make forms more user-friendly while maintaining solid model data on the backend. If you see something we could be doing better, please let us know in the comments.
