===============================
AWS and Django. Our Experiences
==============================+

:author: Kenneth
:status: draft
:category: Django
:tags: django, aws

We recently got to do something we've wanted to do for quite awhile: deploy a fairly complex codebase into the cloud. Our client wanted to use `AWS`_ and we were more than happy with their offerings. There's probably nothing really groundbreaking in this post (what a lead-in, huh!?) but I'm sure some people will find it handy.

Originally, we wanted to launch everything on `ArchLinux`_ but other devs didn't feel it was stable enough and wanted to go with Amazon's provided `Ubuntu`_ images. We didn't really put up much of a fight on it since it makes a lot of sense to use what's provided. Now, if this was our own personal project, we'd probably have stuck with Arch, because the few issues that were initially blamed on Arch ended up being issues with software included with Ubuntu. That said, I'm sure you'll get by with whatever distribution you want to use.

Infrastructure
==============

We probably over-optimized in a few places on the structure of our servers. We have one server (per A and B availability zones) for most of our utility functions (database, `MongoDB`_, `redis`_, `Celery`_, etc). This isn't likely to be required for a lot of projects, especially not when they first go onto the cloud, but we know we'll have to scale this somewhat quickly somewhat soon, so we figure it's better to be over-prepared than under. We do most of our load-balancing in `nginx`_ instead of through Amazon's `ELB`_, but we do use ELB a few places. We also kept using our own mail servers instead of using `SES`_.

One area that infrastructure and our distributed and redundant setup forced us to change our code was in sessions and caching. We normally use `johnny-cache`_ for caching DB queries and johnny-cache uses `memcached`_ for holding the queries and their results. Memcached doesn't replicate, so having a memcached on each app server became somewhat impossible, so we tossed our caching into Amazon's `ElastiCache`_. That worked great until we upgraded our Django to 1.4 to help prevent a strange session bug we had created. Johnny-cache isn't working well in Django 1.4 yet (at least not in our trials) and puked heavily on our optimized API queries. Same story for `django-cacheops`_ and `django-cache-machine`_ (both of which are awesome projects we want to explore more in the future). So we actually ditched query caching for now.

We still needed a cache for `tastypie`_ to use to hold onto it's throttling data. ElastiCache isn't really needed for this because we don't care if all of the app servers keep the same count on our somewhat relaxed throttles, so we just set up memcached on each of those machines. This saves a bit of money on not using ElastiCache and a bit of network traffic.

Replication
===========

We had three major areas of replication: `PostgreSQL`_, MongoDB, and static files (both design-related and uploaded). We had planned on doing multi-master replication with PostgreSQL but after getting advice online from several trusted advisors, we went with a hot-standby streaming replication setup that's actually all built in to PostgreSQL. Not having to install and maintain extra software is always a plus. Setting this up is actually really simple, too.

MongoDB has traditional replication but it's not as full featured as its replica sets, so that's what we went it. Setting these up required a few extra tries (mainly for not having IPs written down and ports opened up) but was ultimately really easy. In fact, later on when we wanted to replace a couple of nodes with a couple of new ones, it was two commands in a ``mongo`` console and opening up ports. We had to run a third machine, a micro instance, as the arbiter since you're required to have an odd number of MongoDB instances, but the arbiter doesn't hold on to any information.

As for replicating static files, we took the lazy way out and just shoved them all onto `S3`_. If we want or need them on another machine, we use `boto`_ and `boto-rsync`_ to move them around. We also use this for pulling files out of MongoDB's GridFS and putting them into S3.
