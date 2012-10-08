=======================================
Getting Started with Django Kickstarter
=======================================

:author: Kenneth
:status: draft
:tags: projects, django, teaching

Recently we wrapped up a `Kickstarter`_ for our `Getting Started with Django`_ video series. I wanted to outline what we've done on the project, what we're going to do, and what it's like to run a Kickstarter.

Kickstarting
============

Starting a Kickstarter takes a lot longer than most people seem to think. I know it took longer than I thought it would. You have to fill out a lot of information about the project: a story, reward incentives, personal information about yourself, like where you're located and tax information. You also have to register and get authorized for an Amazon Payments account, which takes at least 3 days. Even if you had the video, story, and everything else gathered up and created, starting your Kickstarter is going to take around a week to get from sign-up to submission.

Once it's submitted, the Kickstarter crew takes a look at it to make sure you're not doing something against their Terms of Service and that your project meets all the criteria for its submitted category. Since we were doing a technical project, for example, we were required to submit a video as part of our story.

Kickstarted
===========

Once we got everything filled out, submitted, and finally approved, it was just a matter of waiting until the right time to actually launch the campaign. I sat on it for a couple of days, wanting it to run through DjangoCon 2012, but not wanting it to start at DjangoCon. I also had just had my second son, so I was waiting until things calmed down at home a bit, too.

We launch on August 24th in the morning.

Anyone who has sold software or ran a campaign like this will know what I'm talking about, but the anticipation of waiting for the first pledge to come in is hard to deal with. Thankfully, for us, our project was somewhat popular. We had over $2,400 in pledges in the first 24 hours.

Also, after launching it, I was contacted by Jacob Kaplan-Moss about the Django Software Foundation helping to funding the project but with the caveat that there should be free versions of the videos. Anyone who watched the original series will remember that they were all free, and I was more than happy to have a free version. In fact, this got into my head and I decided that if the funds hit $7,500 (my original goal), I'd make them all free, instead of the $10 per video I was originally going to charge. Ironically, by the time the DSF got the funds approved and announced the funding at DjangoCon, we were already over the free video mark.

Finally, the funding ended on September 23rd at a whopping **$13,354**. Way, way more than either of us expected it to get to. I contribute this to a couple of things.

Fundraising
===========

First off, I know that running a campaign during a very closely related conference was a big boost. The Kickstarter was mentioned a couple of different times in the main room of DjangoCon on the first day of the conference and that was one of our biggest donation days. The entire conference, really, was a great donation drive.

Secondly, we did our best to keep it in the public eye. We tweeted about it, mentioned it in comments on blogs and Reddit, and brought it up in the #django IRC channel several times. While this might be a bit obnoxious, it's also the only way to make sure that people who might be interested will see it.

The last thing that we did that really helped, in my opinion, is having smart stretch goals. The first round of contributions, up to $5,000, was just to get the series produced. By adding the first stretch, to $7,500, to make the videos free, gives an easy target for the people who are really invested in the community to drive us toward. Our second stretch goal, $10,000, for doing a second, smaller, series of videos about harder-but-still-common problems was attractive to people that already know the things I'm going to cover in the main series, but know that they still run into problems.

Providing goals that aren't insanely far from your current amount and that enrich the project really helps. And the enrichment factor is a big part, too, as we saw with our last formal strech goal, at $12,000, for doing a round of stickers for everyone that donated $10 or more. While we did eventually reach that goal, it was not met with anywhere near the same enthusiasm as the previous two.

Where we are now
================

So, now that it's over and all the money is collected (minus the fees for Kickstarter and Amazon Payments), we're beginning the serious production work on the project.

We got a new mic and boom arm to provide better audio recording than the original series. The `mic`_ is a RØDE Podcaster with a RØDE boom arm and shockmount to go with it. I also picked up a Western Digital `drive`_ to use as backup and for file transfer when we work physically together on the project.

And, to some people's disappointment, we ordered stickers from `Sticker Mule`_, both with the Getting Started With Django logo and also the Django Pony. They should arrived this week or next and we'll start mailing them out after that. This'll be our first experience with doing a lot of mail distribution, especially to other countries, so if anyone has any advice, we'd love to hear it.

As for the actual content of the project, we've planned out most of what the app we'll be building will do and started on outlining the first couple of episodes. Since this series is about more than just code, we have to take a lot of things into consideration for explaining, for example, how to go about solving your own problems in the Django community (IRC, mailing lists, asking questions intelligently, ``pdb`` and the like, etc).

And that's it for now
=====================

We'll do some more blog posts about individual chapters as we get them under our belts. We also have one brewing about the custom Vagrant base box we're setting up for the project and how you can do one for yourself.


.. _Kickstarter: http://www.kickstarter.com/projects/657368266/getting-started-with-django
.. _Getting Started With Django: http://gettingstartedwithdjango.com
.. _mic: http://amzn.com/B000JM46FY
.. _drive: http://amzn.com/B0041OSQB6
.. _Sticker Mule: http://stickermule.com
