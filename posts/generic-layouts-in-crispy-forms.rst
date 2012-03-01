===============================
Generic Layouts in Crispy Forms
===============================

:author: Kenneth
:category: django
:tags: django, django-crispy-forms, tip

Just a quick tip and sanity check, today, about something I ran into with django-crispy-forms_, the awesome new form library from Miguel Araujo.

This morning, I converted the project we've been building for a client (currently some 1,700 or so files, counting templates, CSS, and icons) from ``django-uni-form`` to ``django-crispy-forms``. It's a pretty painless 
transition, actually. Just do some find-and-replace across your files, basically changing any instance of ``uni-`` to ``crispy-`` (well, and ``form`` to ``forms``), and you're good to go. Then, however, I wanted to 
convert two large forms that we have, which share 90% of their fields, to using the sharable ``Layout`` objects that ``django-crispy-forms`` gives us.

Basically, the forms looked like this:

.. code-block:: django

    class FirstForm(GenericAppFormForTheExample):
        ...

        def __init__(self, *args, **kwargs):
            ...
            self.helper = FormHelper()
            self.helper.layout = Layout(
                "field1",
                "field2",
                "special-field",
                [field3 through field20]

    class SecondForm(GenericAppFormForTheExample):
        ...

        def __init__(self, *args, **kwargs):
            ...
            self.helper = FormHelper()
            self.helper.layout = Layout(
                "field1",
                "field2",
                "special-field2",
                [field3 thorugh field20]

Obviously that was a lot of repetition that we could cut out now that these inheritable layouts exist. By the way, I'm pretty sure this would have been possible in ``django-uni-form`` but likely not as friendly.

First I started off by creating the shared resources. I made two of them since our special fields come in the middle of our layouts.

.. code-block:: python

    form_intro_layout = Layout(
        "field1",
        "field2"
    )

    form_common_layout = Layout(
        [field3 through field20]
    )

And I added them each into my forms. I'm only going to show one form but you'll get the idea.

.. code-block:: django

    self.form_intro_layout = form_intro_layout
    self.form_intro_layout.insert(-1, "special-field")

    self.helper.layout = Layout(
        self.form_intro_layout,
        form_common_layout
    )

And, display-wise, it worked fine. This is, ideally, exactly what you do to extend these layouts. **But**, this causes problems in testing.

We test all of our views, models, and forms. The form tests passed fine, but some of the view tests were throwing up warnings about fields being referenced more than once and, for example, ``special-field2`` missing 
from ``FirstForm``. Obviously something was up with the inheritance and with how I was instantiating objects.

My next step was to stop using ``self`` on the instance-specific versions of the generic layouts and just give them a local name like ``standard_intro_layout`` (the ``standard`` name makes more sense internally and its 
meaning isn't important for this example), but that didn't stop the errors. It did, however, still work fine visually.

I tried using a ``.copy()`` method on the layout, but that doesn't exist. So I turned to the ``copy`` module of Python, specifically the ``deepcopy`` method. I tried with just the standard ``copy`` method, but it had 
the same effect of working visually, but throwing warnings on tests.

The new version looked like this:

.. code-block:: django

    from copy import deepcopy

    standard_intro_layout = deepcopy(form_intro_layout)
    standard_intro_layout.insert(-1, "special-field")
    ...

That works perfectly! It still appears the same visually, like all the methods have, and it also quiets the tests so they pass without warnings.

Unless someone knows of a reason I shouldn't use ``deepcopy``, this seems to be the way to solve this problem. I'm also not sure if it was the ``self`` (which I doubt, since I removed it and the warnings still 
happened), or the fact that both forms extend the same abstract form, or some other variable that led to the problem. Regardless, I'm happy to have solved it.

.. _django-crispy-forms: https://github.com/maraujop/django-crispy-forms
