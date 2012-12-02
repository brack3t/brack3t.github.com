=======================================
Masonry, Infinite Scrolling, and Django
=======================================

:author: Kenneth
:tags: django, javascript
:status: draft

For my current client, we needed a home page that would support a large number of products (it's an ecommerce startup) and, in our first iteration of the new design, deal with content blocks of various sizes. To me, this was a perfect use-case for the Masonry_ jQuery plugin and infinite scrolling, in the vein of Pinterest_. Turns out, this is remarkably easy with Django's ListView_ and the `Infinite Scroll`_ plugin from Paul Irish.

The Django Side
===============

Setting up your view in Django is amazingly simple. We're using the `Pure Pagination`_ pluggable app because we're using Bootstrap_ and want to show their style of pagination with page ranges left out if there are more than a given number of pages (e.g. Pages 1, 2, 3, ..., 9, 10, 11, ..., 17, 18, 19). For the sake of this example, I'll assume you have a template partial that contains your pagination markup. My common naming scheme for this is ``_pagination.html`` in a ``_partials`` directory in my site-wide ``templates`` directory.

So, for our example, we want to show 20 products from our ``Product`` model (creative naming, I know). We set up our view like so:

.. code-block:: django

    from django.views.generic import ListView

    from pure_pagination.mixins import PaginationMixin


    class ProductListView(PaginationMixin, ListView):
        model = Product
        paginate_by = 20

We don't need to specify the template name or the queryset since we want to use the defaults. We add the view to our ``urls.py``:

.. code-block:: django

    from store.views import ProductListView

    urlpatterns = patterns('',
        [...your other routes here...],
        url(r"^products/$", ProductListView.as_view(), name="products"),
    )

And, finally, in our ``store/product_list.html`` template, we render the items:

.. code-block:: html

    {% load humanize i18n %}
    [...other HTML here..]
    <ul class="unstyled wall">
        {% for product in object_list %}
        <li class="brick">
            <div class="thumbnail">
                <a href="{{ product.get_absolute_url }}"><img src="{{ product.image.url }}" alt="{{ product.name }}"></a>
                <div>
                    <h3><a href="{{ product.get_absolute_url }}">{{ product.name }}</a></h3>
                    <p class="price">${{ product.price|intcomma }}<br>
                        {% if product.num_in_stock > 0 %}{% trans "In Stock" %}{% else %}{% trans "Sold Out" %}{% endif %}
                    </p>
                    <div class="description muted">
                        {{ product.description|capfirst|striptags }}
                    </div>
                </div>
            </div>
        </li>
        {% endfor %}
    </ul>
    {% include "_pagination.html" %}
    [...other HTML here..]

As you can see, nothing really special in the HTML. We simply print out the product image, name, price, availability, and description. We include the pagination HTML, too, as we need that for both the Infinite Scroll plugin and to be available for bots and users without Javascript enabled. The only thing left to do is to include the necessary Javascript libraries in your HTML template. I used the jQuery version of Masonry and also included the `Images Loaded`_ plugin to trigger Masonry only after images are loaded to make sure the layout is correct.

The Javascript Side
===================

So with jQuery, Masonry, Image Loaded, and Infinite Scroll all included, it's time to build the small bit of functionality required make this all come together. In your product wall template, or site-wide if you're using this everywhere, either include the following bit of Javascript or stick it into an included file. 

.. code-block:: javascript

    var $container = $(".wall");

    $(function () {
        $container.imagesLoaded(function () {
            $container.masonry({
                itemSelector : '.brick',
                gutterWidth: 25,
                columnWidth: function () {
                    var screenWidth = parseInt(document.documentElement.getBoundingClientRect().width, 10) || parseInt(screen.width, 10);
                    if (screenWidth < 768) {
                        return $container.width();
                    } else if (screenWidth > 768 && screenWidth < 980) {
                        return ($container.width() / 2) - 20;
                    }
                    return ($container.width() / 3) - 20;
                }
            });
        });

        $container.infinitescroll(
            {
                navSelector: ".pagination",
                nextSelector: ".next",
                itemSelector: ".wall .brick",
                loading: {
                    finishedMsg: "",
                    img: "http://pathtoyour.com/loading.gif",
                    msg: null,
                    msgText: ""
                }
            },
            function (newProducts) {
                var $newProds = $(newProducts).css({"opacity": 0});
                $newProds.imagesLoaded(function () {
                    $newProds.animate({"opacity": 1});
                    $container.masonry("appended", $newProds, true);
                });
            }
        );
    });

The first thing we do is cache our selector. We want the ``<ul>`` with a class of ``wall``. Then, when the page is loaded, we add the ``.imagesLoaded`` functionality to the wall. When it sees that all the images in that selector are loaded, it fires off Masonry on the container. I have anything with the class of ``brick`` set as an item and a gutter width of 25 pixels. Then, to define how wide each column is, we do a bit of math on the size of the window. I'm using the same generic(-ish) numbers that Bootstrap uses to define a small/medium/large or phone/tablet/desktop version and how many columns I want in each. I either send back one column, two columns, or three.

The last section loads the ``.infinitescroll`` method onto my container. Within it, I specify that an element with the class of ``pagination`` contains the...well, pagination. And that, within that element, the link that points to the next set of content always has the class name of ``next``. Finally, for ``itemSelector``, I specify that the content on the next page will be anything selected by ``.wall .brick``, which effectively grabs all of the products from the next page.

In the loading section, most of what I'm doing is just cancelling out defaults. I specify an animated GIF to show during loading and set all the messages to blank. In my CSS, I actually hide the animated GIF because, due to how Masonry works, there's no good way to position it at the bottom of the list of elements.

Finally, the function passed as a callback at the end handles what Infinite Scrolling does when it loads the next page of content. We set all of the new products to have 0% opacity, and, when all of their images have loaded, animate the opacity back to 100% and append the products into the existing Masonry layout.

Conclusion
==========

So all of this together, the ListView, the pagination mixin and partial, and the Javascripts, gives you infinite scrolling and a Masonry layout. Sure, it looks a decent amount like Pinterest, but I think that can actually work quite a bit in your favor. It's something people have gotten very used to and it makes sense. One thing we've noticed, though, is that, with very disparate brick heights, your newly-loaded bricks come in and appear out of order. They're still ordered correctly in the source, but may not visually line up. I'll leave that as an exercise for the implementer to make your bricks either equal-height or within a certain range to help prevent that display "bug". Also, page refreshes send a vistor all the way back to the first page, so implementing some ability to automatically jump the user back to where they were in the stack would be a good exercise, too.


.. _Masonry: http://masonry.desandro.com
.. _Pinterest: http://pinterest.com
.. _Infinite Scroll: http://infinite-scroll.com
.. _ListView: http://ccbv.co.uk/projects/Django/1.4/django.views.generic.list/ListView/
.. _Pure Pagination: https://github.com/jamespacileo/django-pure-pagination
.. _Bootstrap: http://getbootstrap.com
.. _Images Loaded: https://github.com/desandro/imagesloaded
