---
layout: post
title: "Hacking couchsurfing.com API"
date: 2013-11-17 20:00
description: "How I hacked couchsurfing.com private API"
comments: true
categories: [couchsurfing, python, api]
---

Traveling is my passion and I'm an huge fan of [couchsurfing](http://couchsurfing.org). Couchsurfing is a global community of travelers, where you can find a place to stay or share you own home with other travelers. On top of that couchsurfing helps you to have genuine traveling experiences while interacting with locals. Here is [how it works](https://www.couchsurfing.org/n/how-it-works). Hell, I've met some of the most awesome people in the world and made lots of friends though this community.

I've hosted a lot of travelers myself, much more than I actually surfed yet. Living in one of the major touristic destinations on the French Riviera, I receive an enormous amount of couch requests (sometimes up to a 10 a day during high season). The problem with couchsurfing.org website is that it doesn't really handle such "high-load" cases properly. There is no information about availability of your couch - when you receive a new couchrequest you can't be sure if you already hosting someone those days or not. There has to be a visual representation of your accepted and pending requests, so you can manage it better. And then, if you make your couch availability public, you can avoid unnecessary couch requests which overlap with days when you are unavailable. To have a better idea what I have in mind, have a look at [Airbnb calendar](https://www.airbnb.com/help/question/447).

<!--more-->

Alright then, I've decided to make an app to solve this issue. First thing I've checked was if there is an API for couchsurfing.org somewhere. It turned out there is no public API available. Here is the response I've received from their support team:

{% blockquote %}
	Unfortunately we have to inform you that our API is not actually public and there are no plans at the moment to make it public.
{% endblockquote %}

Oh well, let's see if we can find a hackaround here.

Fortunately they have an iOS app which uses this private API. I've decided to setup a proxy with [Charles](http://www.charlesproxy.com) to intercept HTTP requests with it. Requests are secured with SSL, but it's not a big deal if you install a root certificate for the proxy on your iPhone. [Here](http://blog.noodlewerk.com/general/tutorial-using-charles-proxy-to-debug-https-communication-between-server-and-ios-apps/) you can find a complete guide on how to setup the whole thing. This way I was able to find access points of their private API and figure out their JSON payload format. Also, thanks a bunch to [Roberto Hidalgo](https://github.com/unRob) for inspiration - he implemented a simple Ruby interface for the same API.

I've decided to design the app so that no data is stored on my servers (I respect the privacy of my users). This means that every time you send a request to the backend (every time you refresh the page or change week/month/year view), a query is sent to my couchsurfing python API wrapper, which sends a couple of requests to couchsurfing API. The first one is a POST to authenticate:

```
https://api.couchsurfing.org/sessions?username=[]&password=[]
```

Then you get a cookie which you can use throughout your session. Another option would be to store users data in the database. Yes, it improves user experience, as you have to wait for the data to be fetched only once, the first time you run the app, and then it is refreshed in the background. But I opted against this option for privacy and storage reasons. The problem with couchsurfing.org API was that there seems to be no way to get all the couchrequests with dates within a certain range, the only option available is to fetch all couchrequests created after a certain day.

```
https://api.couchsurfing.org/users/[user_id]/couchrequests/?since=[unix_timestamp]&expand=couchrequests,users
```

Now, most of the couchrequests are created a week or two, before the planned arrival dates, but occasionally people create those much much beforehand. To take this into account I assumed that no requests are created earlier than 2 months in advance. This is a good enough trade-off between app response time and topicality of the data presented.

Then you just parse the JSON response you get and pass the dates to the calendar javascript. I've stubled upon a quiet curious issue on the way though, with python date representation. Apparently parsing a string into a UTC timestamp is not straightforward at all. One should avoid intermediary datetime structure, as it's not timezone aware. Have a look [here](http://aboutsimon.com/2013/06/05/datetime-hell-time-zone-aware-to-unix-timestamp/) fore more details. Here is one of the ways to do it:

```python

from calendar import timegm
import time

time_string_with_timezone = "2013-06-05T15:19:10Z"

timestamp = timegm(time.strptime(time_string_with_timezone.replace('Z', 'GMT'),
											  "%Y-%m-%dT%H:%M:%S%Z"))
```

That said, I might actually give [arrow](http://crsmithdev.com/arrow/) a shot next time when I need to perform some date/time operations.

Since I'm not a frontend hacker-guru-ninja I've decided to avoid reinventing the wheel and tried to find an html calendar template somewhere. As a huge fan of [Twitter Bootstrap](http://getbootstrap.com/) I've decided to go with awesome [Bootstrap Calendar](https://github.com/Serhioromano/bootstrap-calendar) which served all my calendar visualization needs. It works like that: a custom javascript is loaded for every view (year/month/week/day), this script sends a HTTP requests to a specified URL to fetch the data for a given date range. Thus, I just needed to implement the required route in my web framework (I'm using [Flask](http://flask.pocoo.org/)) to generate JSON, which is being set back to the js library by a HTTP GET.
Also, every time my server gets such a request I have to authenticate the user to be able to access couchsurfing API using python-couchsurfing. With Flask, using basic HTTP authentication (I've used [this](http://flask.pocoo.org/snippets/8/) as a template). Again, I'm not happy about asking for user credentials, as it's major privacy intervention, but there seems to be no other way.

Ah, one more thing. Here is the app:

<http://couchrequests.com>

Check it out (you need a couchsurfing.org account). [Fork it](https://github.com/nderkach/couchsurfing-calendar) on github. Enjoy.

UPD: [Hacking couchsurfing.org API Part 2](/hacking-couchsurfing-dot-org-api-part-2/)

UPD2: couchrequests.com was taken down due to a request from Couchsurfing, Inc, more details [here](http://www.toptal.com/back-end/reverse-engineering-the-private-api-hacking-your-couch)



