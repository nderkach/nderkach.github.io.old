---
layout: post
title: "Hacking couchsurfing.com API Part 2"
date: 2013-11-24 21:51
description: "How I hacked couchsurfing.com private API"
comments: true
categories: [couchsurfing, python, api, heroku, mongo]
---

This is a follow-up for my [previous article](/blog/2013/11/17/hacking-couchsurfing-dot-org-api/). To remind you, I've created a web app <http://couchrequests.com> which extends functionality of <http://couchsurfing.org> by accessing their private API.

I've decided to go an extra mile and implement another highly-demanded feature: couch availability calendars embedded in users' profiles. Wouldn't it be just awesome to have such a calendar which is updated automatically?

So I'd done some research to see which embedding options are actually available in couchsurfing.org profiles. By trial and error I figure out you can use either bbcode or a restricted set of html tags. Luckily ```<a>``` and ```<img>``` tags were available. So here is a solution I came up with: generate a JPG image with my service and embed this link into a profile.

Then I figured I could create a page for every user which displays a calendar for the current month. This page would be public and requires no authentication. Then I could make a screenshot of it and pass it through to a couchsurfing.org profile. Now, I want those snapshots to be more or less up-to-date, but no real-time updates are required. I assumed that most of the users would be satisfied with a refresh interval of one day.

To make those screenshots I've decided to use [Website Thumbnail API from immediatenet.com](http://immediatenet.com/thumbnail_api.html). The API is completely free and caches images. This is great, since it diminishes backend load, avoiding unnecessary backend queries on every profile request. The downside is that an image is cached up to 8 days, so the image displayed in a profile could be not so up-to-date. When a user clicks on it, he is redirected to an app page, which is refreshed every day though.

Another potential issue with embedding images is browser caching, but this shouldn't be an issue as the image is typically going to be updated on the next browser session, and most of the time the host's profile is seen only once by a given user.

Then, I need to find a way to generate individual user pages like that. Since I can't make queries to the API anymore, I need to store all the couchrequests's dates in the database. As I'm storing JSON I've decided to go with a document-based DB, with an index built on user ids. I've opted for good old mongo from <http://www.mongohq.com> for this purpose.

I've made two options available for users:

* Create a snapshot of the current view (week/month/year) "on demand"
* Automatically regenerate the snapshot

The second option is particularly appealing as it implies that once a user enables it, his calendar will always stay up-to-date and visitors of his profile will always see his couch availability information, with no action required from him. A user embeds the code provided by the application in his profile once and he is set.

I need to be able to send API requests on behalf of a user. To make it happen I'm storing user ids mapped to session cookies in the database. Luckily those session cookies have an unlimited expiration time (but If they didn't I'd just send an occasional keep-alive request).

Now, to refresh the calendar dates, I'm running a cron-like job every day with [Heroku Scheduler](https://devcenter.heroku.com/articles/scheduler). I iterate through the db and check if a document has a "cookie" node  with `find({"cookie", {$exists: true} } )`

Users have an option to opt-out from storing their sessions as well.

Also, for those calendar pages I'm making a screenshot of, I've decided to go with a more minimalistic design, as the resulting image embedded into a profile would be rather small, so readability is quiet important. Here is how a user page looks like: <http://www.couchrequests.com/4732279>

Finally, whichever option you'll choose, you'll get a nice popup with the code:

{% img /images/embed_code.png %}

So, here you go, I hope you'll enjoy using the [couch availability calendar](http://couchrequests.com) and I'm open for suggestions for new features. Also, the code for the app is available [on github](https://github.com/nderkach/couchsurfing-calendar).

PS: here is how it looks in my profile:

{% img /images/couchsurfing_calendar.png %}