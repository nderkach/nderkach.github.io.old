---
layout: post
title: "Things I've learned porting Ebay SDK to Python 3"
categories: [python, python3, ebay, sdk, curl]
comments: true
---

One day I've decided to play around with Ebay a little bit (nothing illegal i swear). And since I love python my first step was to find if there is an Python SDK available somewhere. And indeed there was [one](https://go.developer.ebay.com/developers/ebay/product-updates/announcing-ebay-open-source-python-sdk). Had a look at the corresponding [github repository](https://github.com/timotheus/ebaysdk-python) it seems it's being maintained as well. Great! Well, not really - it doesn't work with Python 3. Oh well, time to get my hands dirty.

First thing I did was to run an infamous [2to3 tool](http://docs.python.org/2/library/2to3.html). And frankly it was quiet a help as it's had a great work, especially with those tedious brackets around print statements (but unfortunately not in doctests).

Well the "porting" part itself was pretty smooth and straightforward in fact. The problem was that, well the SDK itself was mostly broken and I had to fix up quiet a lot on my way through.

Luckily there is an [API testing tool](https://developer.ebay.com/DevZone/build-test/test-tool/), which I could use as reference to form HTTP requests with correct headers and body XML.

Then I've stumbled upon a curious issue with pycurl and io.StringIO when using StringIO as a write callback for curl.
The error looks as follows:

{% codeblock %}
pycurl.error: (23, 'Failed writing body (1457 != 1460)')
{% endcodeblock %}

Here is what I've found in documentation:

{% codeblock %}
CURLE_WRITE_ERROR (23)

An error occurred when writing received data to a local file, or an error was returned to libcurl from a write callback.
{% endcodeblock %}

That's quiet an ambiguous description right here. Meaning that the issue could be either with a StringIO object or with my implementation of the write callback. I've tried all the variations. Originally I just tried the following

{% codeblock lang:python %}
self._curl.setopt(pycurl.WRITEFUNCTION, self._response_body.write)
{% endcodeblock %}

where ```self._response_body``` is a StringIO. This failed. Then I've tried to add a lambda as a write callback:

{% codeblock lang:python %}
self._curl.setopt(pycurl.WRITEFUNCTION, lambda buf: self._response_body.write(buf))
{% endcodeblock %}

Nope, didn't work either. No what did work was to add a real function, such as:

{% codeblock lang:python %}
## Callback function invoked when body data is ready
def body(buf):
    self._response_body.write(buf)

## Callback function invoked when header data is ready
def header(buf):
    self._response_header.write(buf)
{% endcodeblock %}

Well, frankly I could find an explanation why it worked this way only. I've posted the question on [stackoverflow](http://stackoverflow.com/questions/19606447/pycurl-and-io-stringio-pycurl-error-23-failed-writing-body) and I filed a [bug for pycurl](https://github.com/pycurl/pycurl/issues/51) as well. Will see...

Anyway, if you wish to try out python 3 friendly ebay sdk, [get it here](https://github.com/nderkach/ebaysdk-python/).

