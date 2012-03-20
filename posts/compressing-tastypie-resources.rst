==============================
Compressing Tastypie Resources
==============================

:author: Kenneth
:status: draft
:category: Django
:tags: django, tastypie

Django has a great little API library out there called `django-tastypie`_. We've mentioned it before, in our `post about AJAX`_, but didn't go over how to really use it. We won't go over all the ins and outs in this post, either. Mostly just wanted to go over a handy little tip if you're using Tastypie with related resources but only need to hold on to a small part of the related records.

Models
======

For the purposes of this blog post, we're just going to set up two models. The first will be pretty much any model you want to imagine, but we'll use an author.

.. code-block:: django

    class Author(models.Model):
        name = models.CharField(max_length=255)
        birth_date = models.DateField(blank=True, null=True)
        bio = models.TextField(blank=True, default='')

Like I said, this could be any model in your system. The second one could be, too, but it has to relate to the first. We'll follow the usual track with authors and make a model for their books.

.. code-block:: django

    class Book(models.Model):
        title = models.CharField(max_length=1024)
        publication_date = models.DateField(blank=True, null=True)
        summary = models.TextField(blank=True, default='')
        author = models.ForeignKey(Author, related_name="books")

This sets up a really simple model that relates with a many-to-one relationship back to our original model.

Resources
=========

If you haven't worked with Tastypie before, the basic idea is that you create resources, usually ``ModelResource``s for each of your models that you want to share through your API. You can create multiple APIs if you want, perhaps with different authentication/authorization schemes or to hold on to different parts of your app. We use this concept for some of our clients to we can have different APIs on the front- and backends. We need to create resources for both of our models.

.. code-block:: django

    class AuthorResource(ModelResource):
        books = fields.ToManyField(
            "library.api.BookResource",
            "books", full=True)

        class Meta:
            authentication = Authentication()
            authorization = Authorization()

            queryset = Author.objects.all()
            resource_name = "authors"

            filtering = {
                "id": ["exact"],
                "books": ALL_WITH_RELATIONS,
            }

        def dehydrate(self, bundle):
            bundle.data["titles"] = bundle.obj.books.values_list("title")
            return bundle


    class BookResources(ModelResource):
        author = fields.ForeignKey("library.api.AuthorResource", "author")

        class Meta:
            authentication = Authentication()
            authorization = Authorization()

            queryset = Book.objects.all()
            resource_name = "books"

            filtering = {
                "author": ALL_WITH_RELATIONS,
                "id": ["exact"],
            }

Both of these resources are pretty simple and work just fine. 

.. _django-tastypie: http://tastypieapi.org
.. _post about AJAX: http://brack3t.com/ajax-and-django-views.html
