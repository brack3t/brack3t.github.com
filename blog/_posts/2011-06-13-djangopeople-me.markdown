---
layout: blog
title: Djangopeople.me
---

What is it?
-----------

At it's core, [djangopeople.me](http://djangopeople.me) is a place for [Django](https://www.djangoproject.com/) developers to register and be found based on their geographic location, whether it's by recruiters or 
a lonely Django developer like ourselves trying to find others close to him/her. That's the general idea behind the site.

We wanted to make it clean, simple and fast. For the maps we ended up using [Leaflet's](http://leaflet.cloudmade.com/) open-source Javascript library. We initially started with Google Maps, but the rate limiting was 
too restrictive on the geocoding API for our needs and frankly, loading maps seemed very slow. So, with the project being free-to-use (and us being cheap), we looked for something else.

Yahoo! Maps was the next step as they had a decent API and allowed more requests. Google currently allows 2,500 requests per day and Yahoo allows 5,000 requests (geocoding). Yahoo's maps were faster to load but we seemed to have nothing but issues with [Google Chome](http://google.com/chrome) throwing a YMAP Javascript error we could never track down. We didn't think it would be all that awesome telling Chrome users to clear their cache up to three times before the maps would load again (If you've had this issue and a have fix, for the love of all that is holy, let us know about it please).

After searching around for a day, we came across Leaflet. It was simple to implement and the maps were extremely fast to load compared to Google and Yahoo. You could also make the argument that they don't have nearly 
the traffic that Google and Yahoo have but Leaftlet was simple and it worked. Also, our scientific method for testing the load times was simply us reloading pages over and over as we worked on functionality. So don't 
expect any charts. Chris is an impatient guy, so if he gets annoyed by something he's built, it's not going live.

There is one issue we ran into with Leaflet and it has yet to be resolved. Their geocoding API, after about two weeks of use, started retuning funky results. By funky I mean, resolving *Las Vegas, NV*, to Livingston, Illinois. *Dallas, TX* would come back as being in Australia. Chris emailed their support team and they have confirmed it is a bug but they do not have an idea of when it will be resolved. Aside from that, their service has been fantastic.

It's already been done.
-----------------------

This is true, [djangopeople.net](http://djangopeople.net) was the inspiration for building djangopeople.me. We realized that djangopeople.net had been down for some time, intermittently. It was useful and we had both 
been contacted by people looking for Django developers through that site. So the original idea was to bring that functionality back and hopefully make it better.

Kenneth came across a posting on [Convore](http://convore.com) when we were almost ready to launch the site. It appeared that a group of developers were getting together to bring djangopeople.net back online. We almost 
stopped working on the site as dp.net came back online for a short time. It was really slow and then ended up going back down a day later. So we decided to finish the small bug fixes we were working on and launch it.

So, just to be clear, djangopeople.me was inspired by djangopeople.net. We just wanted to bring that back to the Python/Django community. We believe the interface is better and faster. Also, using [Twitter](http://twitter.com) 
as the main auth system made it dead simple to sign up. We've had a few people ask for other authentication methods but at the moment Twitter provided exactly what we wanted with no hassle.

We use your Twitter username so it's very easy to find a profile of someone you already know and follow. This also makes it a little difficult to add other authentication methods as we'll now have to deal with allowing people to choose usernames and possibly picking a Twitter username that a future developer would sign up with. We're open to suggestions but we like the simplicity of using Twitter. Having recently wrapped up a project using the [Facebook Python SDK](https://github.com/facebook/python-sdk), it was nice for Twitter's auth system to just work and not get in the way.

What's next?
------------

We've had a good amount of people sign up and we've gotten some good feedback. We have a couple of ideas for tying into other Django services but we're waiting for replies to our emails at the moment. We both 
contract/freelance full time, so we'll probably spend the next week or two throwing ideas around before we know what the next step is. In the meantime, check out the site and let us know what you love and/or hate about 
it. We've got thick skins.
