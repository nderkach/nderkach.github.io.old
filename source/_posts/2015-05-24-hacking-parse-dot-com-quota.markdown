---
layout: post
title: "Hacking Parse.com quota"
date: 2015-05-24 11:41:37 +0300
comments: true
categories: [nodejs, javascript, parse, balancer, quota]
description: "Hacking Parse.com quota"
---

Parse.com is a great BaaS (backend as a service) solution, it helped me numerous times to take a mobile app prototype or MVP off the ground and focus on the product features, rather than building the database layer, push notifications infrastructure and analytics platform from scratch.

The provide you with a pretty lean pricing structure when you pay proportionally to the number of requests you make to the API on top of free 30 requests per second. Now, even though one should optimize an app to make as few requests to the API as possible, Parse has no built-in request throttling support, i.e. if you have a spike of API usage at some point, there is no way to balance it. If you reach your plan quota, all the requests made above the threshold are being simply rejected. Parse seem to present it as a feature ensuring a better QoS and recommends upgrading your quota: this way the API is respnsive and you don't have to wait, but frankly I'd much rather make my users wait a little bit every once in a while than lose their data.

<!-- more -->

Naturally, if you consistently exceed your plan's quota, you should upgrade. In cases of short request bursts - here is the solution.

# Queue'em Up!

For starter, I've wrote a simple script sending 50 requests/s to Parse API, and see how it reacts. It seems that the quota limit is checked every minute, so on my free plan I can make up 900 requests a minute. When you reach your quota limit, you get a ```400```.

I've built a very simple proxy with Node.js which intercepts all the requests to Parse.com and puts them in a throttled FIFO queue. Every 30 seconds or so, a requests is taken from a queue and sent to ```api.parse.com```.
No more request drops!

Here is the source code for the proxy: https://github.com/nderkach/parse-balancer

Feel free to try it out live here as well: https://parse.space
I pinky promise not to read your payloads. Really.

To make for example iOS SDK work with our brand new proxy, a simple string replacement in Parse binary will do the trick:





