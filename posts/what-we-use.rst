===========
What We Use
===========

:tags: workflow, tools
:author: Kenneth
:status: draft

We've noticed several people publishing lists of what they use to do their work lately, so we thought we'd join in.
Below are the tools we use nearly every day to do our work. If you don't know, our work is the entire stack,
from setting up servers, to Django_ and Python_ programming, to front-end web development and design. Obviously,
with such a large area of work, we end up using a lot of different products and libraries, so, below is a list of our
most common items.

Testing post recieve hook.

.. contents::
    :depth: 1

VIM
===

Do I really need to say anything about VIM? It's an amazing text editor and handles everything we need it to handle, from Django to HTML to LESS_ to Javascript and these blog posts (which are all done in RestructuredText_ and published with Pelican_). Yes, it has a steep learning curve but that can be pretty easily mitigated by starting with a GUI version (like gVim_ or MacVim_) and mapping controls to settings you're used to. Use nothing else for a week or two and you'll be back to your old speed, if not faster.

There are a few plugins that we want to highlight, too.

Plugins
-------

First up is Ctrl-P_. It's a fuzzy file finder that'll search your tree. One of the handiest features is the ``ctrlp_working_path_mode`` setting when it's set to ``2``, which makes the search start at the nearest ancestor with a version control file/folder in it, such as the ``.git`` folder. It will ignore files you tell it to ignore and can search based on most-recently-used or through ctags, which would let you find a file based on classes or methods it contains.

Next is the complementary plugin, NERDTree_. We don't use it as much as we used to, now that we've switched to Ctrl-P, but it still gets a fair amount of usage. It's basically equivalent to TextMate or Sublime Text 2's project browser feature. Not much to explain here, but we do end up spending a fair amount of time in it.

We also use Buffergator_ a lot since we tend to jump between a lot of files throughout the day. Buffergator lets you quickly see all of your buffer, sorted in a couple of different ways (MRU and buffer number are the two modes we use most) and will let you preview a buffer without switching to it, which is pretty handy when you find out that all of our apps contain two view files named ``backend.py`` and ``frontend.py`` and two urls files named the same and, potentially, two files of APIs named the same too (we like naming schemes, OK?).

For eye-candy, we also love the Powerline_ plugin. It gives an awesome statusline effect and makes you feel cool, which is so rare for us nerdy types. Right, guys? Right?

Since we write a lot of different languages, having syntax checking is really important. We use Syntastic_ to check all of our files for erros, and that requires us to have PyFlakes or PyLint installed system-wide, for checking Python. We also install Tidy (from Homebrew) for checking HTML and gjslist (Google's JSLint) or the standard JSLint installed for checking Javascript.

Firefox
=======

I know, I know, Chrome is the big browser on the block. And Safari is master of the mobile arena. But we both really like Firefox. Two main reasons, I think. One, Firebug_ is still the best inspector on the market, at least in my experience (and the Web Developer Toolbar's option to turn off cache isn't replicated in Safari at all). Secondly, Pentadactyl_ is unmatched in any other browser, bar none.

That said, we both use Chrome and Safari for testing sites, holding on to multiple logins, and testing out Socket.IO and the like. We just use Firefox for standard browsing.

Chris doesn't make use of it, but I definitely use the hell out of Firefox's tab groups, too. If you haven't tried out this feature, hit ``Cmd-Shift-E`` (or likely ``Ctrl-Shift-E`` for you PC types) and check it out. Amazingly handy when you work on a lot of projects or need to keep several browsing trains of thought running at the same time. I really wish they had a way to unload a group of tabs from memory, though. That's all it's missing.

Addons
------

As mentioned, we both use the hell out of Firebug. We also use Pentadactyl a lot. You all should know Firebug, so I'll talk about Pentadactyl for a bit. Since we're both big VIM users, Pentadactyl lets us use our keyboards for most of the browsing we do. We can open new tabs with ``t`` or just go to a new URL in the current one with ``o``. We both set DuckDuckGo_ as our default search engines, too, so a quick ``t`` followed by ``!django <search term here>`` lets us quickly look up documentation.

I mentioned the `Web Developer Toolbar`_ earlier too. We don't make extensive use of all of its features, but we do use the cache disable and browser resizing features. Before Firebug and some other addons were around, though, it was a killer addon to have for its ability to inspect CSS.

The last addon that we made a lot of use of is JSONovich_ which enables a pretty display of JSON in Firefox. If you end up having to look at a lot of JSON all day long (like, say, when you're working with Tastypie), it's really useful to have a nice view.

iTerm 2
=======

The terminal that comes with Mac OS X Lion is great, but we both had some display weirdnesses when it was coupled with Tmux and our VIM setups. iTerm_ didn't have any of those issues (but it did bring around a new one with the pastebin). It looks good, works well, and supports native splitting. The newest version introduced support for a custom version of Tmux, but neither of us have tried that out yet. If/when we do, we'll probably write something about it.

Tmux
====

Ah, Tmux_. If you don't know what it is, you should. It's a terminal multiplexer with is a long way of saying it lets you split one terminal window into lots of terminal windows, both as whole new windows (groups of panes) and individual panes. We end up running several servers to do our work, so being able to see and control all the servers in one Tmux window is amazingly handy.

I use Tmuxinator_, too, to automate the creating of windows and panes for projects. You write a YAML file with rules for what windows and panes you want and what you want ran in each pane, then run ``tmuxinator start <project name>`` and you're up and running.

Also, when working on servers, Tmux makes it really easy to attach to an existing session and you can watch or work in tandem with another developer.

Homebrew
========

Homebrew_ is probably something we don't even need to mention. It's like ``apt-get`` or ``pacman`` for your Mac, which means it'll let you install UNIX and Linux packages for your Mac through an easy command-line interface. You can even, with a bit of tinkering, install libraries that already come with your Mac, so if you need that bleeding-edge version of Ruby or Python, it's simple to install or uninstall.

We've written a couple of Homebrew recipes for internal use at our main client and that has definitely helped with keeping the team using the same packages and versions, regardless of machine or technical aptitude.

Skype
=====

I won't bother to link to Skype, I'm sure you already have it installed. We use it all day every day, though, since Chris and I pair program probably 98% of the day. Skype is how we conduct almost all of our client meetings and we talk through it all day ourselves. It's not a perfect system but it works really well and is less taxing on our machines than Google Hangouts.

Screen Sharing
==============

Yes, the built-in Screen Sharing on your Mac (go find it in your Finder, it's at ``System > Library > CoreServices`` and stick it in your Dock). This, combined with Skype, a VPN or DynDNS url, and a couple of opened ports and you have a quick and easy way to do pair programming remotely. Set it to start in Observe Mode and you don't have to worry about disturbing the person you're working with either.

DynDNS
------

Chris and I each have a free domain name with DynDNS_ that we point to our machines through our routers and open up for web hosting (port 80) and screen sharing (the typical VNC ports). That way we can just point screen sharing to the URL and we're connected, no remembering IPs or having to look them up through the Network control panel or the terminal. Not a ground-breaking hint by any means but definitely one that has saved us time.

Balsamiq
========

We use Balsamiq_ for mocking up interfaces before we build them. Most of our work currently is replacing existing interfaces, used by a fair-sized team for years, with new, better, smarter ones. We plan out most of them before we start to build them and vet them with the people that will be using them. Balsamiq gives us a great, low-fidelity version we can show people that doesn't lead them to make too many assumptions about appearance or advanced functionality.

Favorite libraries and frameworks
=================================

As far as favorite frameworks go, I'll give you three guesses which one is our favorite and the first two guesses don't count. We use Django for pretty much everything and haven't seen a reason to move away from it yet. But we don't live or work in a vacuum, so we've had to add a lot of other items to our repetoire.

Nginx_ is our web server of choice. It's fast, lightweight, and really easy to set up. We'll write up our crazy workflow with Nginx, Lua, Redis, and Scrapy soon.

For DOM manipulation, we use jQuery. It's solid and dependable.

For Javascript MVC, we use BackboneJS_. It may not be the newest or most fancy of the MVC frameworks, but we've come to understand it somewhat well, it works reliably with Tastypie_, and uses the awesome Underscore.js library. Sadly, we haven't been able to make extensive use of Backbone yet, but we hope to change that in the near future.

Last but not least, for design, we both definitely love the Twitter Bootstrap_ collection. It's a great starting point for web design and really helps you get to creating the product instead of worrying about what it looks like.

Summary
=======

I think that pretty much covers the tools and libraries we end up touching in our day-to-day work. Our work and requirements are constantly changing, though, so this post might be one that we need to revist every six months or year. If you have any suggestions for new products to check out, or ways we can make our current favorites even better, let us know on Twitter or in the comments below. Thanks for reading.

.. _Django: http://djangoproject.com
.. _Python: http://python.org
.. _Ctrl-P: https://github.com/kien/ctrlp.vim
.. _NERDTree: https://github.com/scrooloose/nerdtree
.. _Buffergator: https://github.com/jeetsukumaran/vim-buffergator
.. _Powerline: https://github.com/Lokaltog/vim-powerline
.. _Pentadactyl: http://dactyl.sourceforge.net/pentadactyl/
.. _Firebug: http://getfirebug.com/
.. _Web Developer Toolbar: https://addons.mozilla.org/en-US/firefox/addon/web-developer/
.. _JSONovich: http://lackoftalent.org/michael/blog/json-in-firefox/
.. _iTerm: http://www.iterm2.com/
.. _Tmux: http://tmux.sourceforge.net/
.. _Tmuxinator: https://github.com/aziz/tmuxinator
.. _Homebrew: http://mxcl.github.com/homebrew/
.. _DynDNS: http://dyndns.org
.. _Balsamiq: http://mxcl.github.com/homebrew/
.. _Nginx: http://nginx.org
.. _BackboneJS: http://backbonejs.org
.. _Tastypie: http://tastypieapi.org
.. _Pelican: http://pelican.readthedocs.org/en/2.7.2/index.html
.. _RestructuredText: http://docutils.sourceforge.net/rst.html
.. _Bootstrap: http://twitter.github.com/bootstrap
.. _LESS: http://lesscss.org
.. _DuckDuckGo: http://duckduckgo.com
.. _gVim: http://www.vim.org/download.php#pc
.. _macvim: https://github.com/b4winckler/macvim
.. _Syntastic: https://github.com/scrooloose/syntastic
