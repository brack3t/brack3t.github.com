=================
Semantic Snakeoil
=================

:date: 2011-10-03 21:31
:author: Kenneth
:tags: semantics, html, css
:category: web development
:slug: semantic-snakeoil

For years now, we've been told that we should always use semantic class names and IDs in our HTML. 
And, in general, this is a good practice, I don't want anyone to think it's not. You *should* make 
your markup, all aspects of it, as accurate and related as possible. The problem, though, comes 
when people become fanatical about their classes and IDs to the point of swearing off grid systems 
and CSS frameworks in the name of "proper" web development practices.

So why isn't it a big deal? Both the `W3C <http://www.w3.org/TR/html5-author/global-attributes.html#classes>`_ and the `WHATWG <http://developers.whatwg.org/elements.html#classes>`_ emphasize that the class attribute, and its values, if specified, should describe the nature of the content and not the presentation of it. And for the most part, I agree. This ideal has limits, though, that need to be addressed and understood. **It's currently impossible to be 100% semantic and still be useful.**

Let's start with the argument presented by the idealists: All class names and IDs should be semantic and address the nature of the element, not its display. This ends up with us needing to create new classes and ID for every project we do, tied directly to the use case of each bit of markup. Yes, some things could be repeated and copied from project to project, but a decent majority of the CSS would be unique. This is great if you're the only person working on something, or working with a small group that works together on many projects. This is rarely the case with large or open-source projects, though.

Now, let's look at the argument from the utilitarians: All class names should be generically semantic to the structure of the document and relate to the use case and nature of the element; IDs should be semantic to the project or page. This opens up the project to third-party grid systems and the like. So long as the class names have meaning to the document, even if it's not semantic meaning to the content it's wrapped around, it's still valid and logical.

For a second argument against made-to-order frameworks for each project: using third-party libraries like the awesome `Twitter Bootstrap <https://twitter.github.com/bootstrap/>`_, newcomers to the project will either have experience with the library, or can refer to its documentation to get up to speed; if you're using a custom library of CSS for the project, a new developer needs to familiarize themselves with the whole thing with, lilkely, little to no documentation.

To my mind, and many other professionals, the point of the ``class`` attribute is to describe the element or its content. The element, though, is the more important bit to describe since using a semantic element already describes the content. If you put a list of links into a ``<nav>`` element, there's no reason to give it a class of ``nav`` or ``menu`` or the like. There is, however, plenty of reason to give it a class that describes how it relates to other elements, such as ``span6``.

Using these systems, as well, gives us the ability to reflow content in a way that remains semantic. To set the stage, we have a ``<section>`` of content that is the main content of the site, and an ``<aside>`` that is our sidebar. We want the page to reflow based on media queries.  If we use an idealistic set of classes, such as ``sidebar``, once the page reflows and the ``<aside>`` ends up below the main content, its class no longer has the same semantic meaning. Yes, we could have used a class like ``secondary-content`` or the like, but few people seem to use class names like this. If, though, we use something like ``span6``, it loses no semantic meaning after the reflow as it is still spanning 6 of our columns. Yes, that's describing the display but it's describing it on relation to all the other elements, so it is semantic in context.

The other point I keep seeing being made is, that, to change the width of an element, the designer has to go and edit the HTML. If, though, you're using HTML correctly, to where classes describe more than one object, if I want only one instance of an object to be wider, I have to either add, remove, or edit a class in the HTML anyway. If your knee-jerk reaction is that "my classes aren't set up that way" and you're using classes like they're IDs, to where they're unique, you aren't using them correctly and should be using IDs on your elements. This also ties into more serious programming; if your model changes, you have to (usually) update your views/controllers and templates/views. This isn't a unique concept and shouldn't trip up any serious developer or designer.

Search engines don't seem to care about class values, either, so using more of them won't hurt your site's SEO (if you care about that). Also, as far as content being available, if people are scraping your HTML, it would be better to offer it in a more consumable format like an JSON API or an RSS feed (aka XML). This way the content is separated from your presentation of it and can be used however the consumer needs it.

So, I put forth a new definition for classes:

    There are no additional restrictions on the tokens authors can use in the class attribute, but authors are encouraged to use values that describe the nature of the content, rather than values that describe the desired presentation of the content. **It is also valid to use values that describe the element.**

Have comments? Continue the discussion on `Hacker News <http://news.ycombinator.com/item?id=3072260>`_.
