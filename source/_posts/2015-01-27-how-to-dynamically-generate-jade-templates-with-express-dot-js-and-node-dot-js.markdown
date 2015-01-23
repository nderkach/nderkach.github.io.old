---
layout: post
title: "How to dynamically generate Jade templates with Express.js"
date: 2015-01-27 20:22:48 +0800
comments: true
categories: [nodejs, javascript, expressjs, jade, templates]
description: "How to render Jade templates on demand with Express.js and Node.js"
---

Recently I've been playing with Express.js - yet another MVC web framework. Even though I'm not particularly fond of Javascript for the backend, I decided to give it a shot for yet another [weekend project](https://github.com/nderkach/dropout). Even though I had to learn a new framework (Node.js with Express.js) and a new template language (Jade), my journey to building a simple website that does some web-crawling and serves as intermediary for passing through HTTP requests was fairly straight-forward. In fact, I enjoyed working with Jade, I definitely prefer it to a more verbose Jinja2.

The website works like this: you search for a term and it displays a list of videos matching the criteria. The search button sends an AJAX call to the backend which automagically generates a dictionary with metadata and links to the videos. Here comes a gotcha: instead of returning a JSON, you render a template on the backend side and return it to the client.

<!-- more -->

Say, you have a container ```#search-results``` to display your results and you install the following event handler for the search query:

```javascript
$(function() {
  $('#form-submit').click(function(e) {
    e.preventDefault();
    $.get("/search?q=" + $('#search-field').val(), function(response) {
      $("#search-results").empty()
      $("#search-results").append(response)
    });
    return false;
  });
});
```

Now, you need to render required html with your search results on the backend.
All you need to do is create another view ```search.jade``` with your template.
Then on the backend, you add a route for ```/search``` which handles the request:

```javascript
router.get('/search', function(req, res, next) {

  var query = req.query.q;
  var videos = {};

  // do some video magic here

  res.render('search', {'result' : videos});
});
```

Voila! Now you call you backend with a query and it renders your search results, which are dynamically added to your client's DOM.


