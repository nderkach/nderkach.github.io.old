---
layout: post
title: "Using Flickr API for beautiful landmark photos"
date: 2015-01-20 14:01:44 +0800
comments: true
categories: [api, flickr, ios]
description: "Using Flickr API for beautiful landmark photos"
---

While building [Traffle](http://angel.co/traffle) (a social travel iOS app), I've stumbled upon a problem: how to generate high-quality landmark photos for a particular destination. To give you an idea of the kind of pictures I was looking for, check out [Yahoo! Weather](https://mobile.yahoo.com/weather). In Yahoo! Weather image content is curated: the best photos are being handpicked by editors.

Tha app required to create a simple algorithm which produces relevant content. Surely no algorithm can compete with curated content, and I didn't want to spend time implementing a machine learning algorithm, like [this one](http://www.academia.edu/4610174/Exploiting_Flickr_Tags_and_Groups_for_Finding_Landmark_Photos).

Playing around with [flickr.photos.search](https://www.flickr.com/services/api/flickr.photos.search.html) I empirically found the optimal combination of accuracy, tags and [interestingness](http://www.steves-digicams.com/knowledge-center/how-tos/online-sharing-social-networking/what-is-flickr-interestingness.html). Here is the API request I came up with:

```http
GET https://api.flickr.com/services/rest/
?accuracy=11&
api_key=API_KEY&
lat=37.7833&lon=-122.4167&
method=flickr.photos.search&
sort=interestingness-desc&
tags=scenic,landmark&
api_sig=API_SIG
```

If you are building an iOS app, I recommend [this project](https://github.com/justinmfischer/core-background) to build a nice landmark slide show like this one:

![Animation Example](http://cl.ly/image/0W3V3b281g0M/13343.mov.gif)


Surpisingly, a single call to Flickr API can solve a problem good enough and seems to be a resonable compromise. Yes, it's not perfect, and as much of a perfectionist I am, I think good engineering relies heavily on compromises - good enough now is better that perfect tomorrow. I'd argue that selecetion of landmark photos is a very subjective process in iteself, and it could be that the imperfections of this approach actually add a more human touch to the photos. In case of San Francisco, you'd not only see the Golden Bridge and Alkatraz, but also ordinary people in coffeeshops and backalleys.