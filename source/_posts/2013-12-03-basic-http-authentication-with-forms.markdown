---
layout: post
title: "Basic HTTP Authentication with forms"
date: 2013-12-03 10:21
title: "How to make Basic HTTP Authentication look nice"
comments: true
categories: [http, authentication, flask]
---

Here is a fairly simple trick, how to use basic HTTP authentication (yes, the one where you have an ugly popup window) with HTML forms. I'm actually quiet surprised not a lot of developers are aware of it, as it's quiet neat.

In short, the idea is that you create a form and pass authentication data to your backend.
Simple enough, right? Here is how it's working under the hood.

You create a basic HTML for with a submit button:

```html
<form id="form-signin">
	<input type="text" id="login" placeholder="Login or Email" required autofocus="">
	<input type="password" id="password" placeholder="Password" required>
	<button type="submit"> Sign In </button>
</form>
```

On top of that you add a javascript handler (this one uses jQuery), which sends a XMLHTTPRequest to the backend authentication folder (`/login` in our case) via username and password keys in the ajax call:

```javascript
$(document).ready(function()
{
	$('.form-signin').submit(function()
	{

		var username = $('#login').val();
		var password = $('#password').val();

		console.log("login: " + username);
		console.log("password: " + password);

		$.ajax(
		{
		  'username' : username,
		  'password' : password,
		  'url'      : '/login',
		  'type'     : 'GET',
		  'success'  : function(){
		  	  /* authentication ok, redirect to the app */
		  	  window.location = '/';
		  },
		  'error'    : function(){
		  	  /* bootstrap alerts */
		  	  $('#alert').remove();
			  $('<div class="alert alert-danger" id="alert">Wrong login/password. Try again.</div>').insertAfter("#password").hide().slideDown("slow");
		  },
		}
		);
		return false;
	});
});

```

I won't go into details, how Basic HTTP authentication works, but the idea is that the server sends a request for authentication as a 401 response with `WWW-Authenticate` header. So, whenever your backend receives a HTTP request with no authorization header, you want to send this response to authenticate user.

Then the client sends another request with authorization this time, such as:

`Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==`

Where the token are concatenated login and password in Base64. This is why you absolutely must use Basic Auth over HTTPS, if such option is not available, stay away from it.

Finally when you backend receives the credential and you check that they are valid, you send another response (e.g. 200 for success and 400 for failure). That's it.

As a bonus, here is a backend implementation of the above in Python with Flask (based on [this](http://flask.pocoo.org/snippets/8/) snippet:

```python
from functools import wraps
from flask import render_template, jsonify, Response, abort

def check_auth(username, password):
    """This function is called to check if a username /
    password combination is valid.
    """
    try:
    	# check if username and password are correct
    except AuthException:
    	return False
    return api

def authenticate():
    """Sends a 401 response that enables basic auth"""
    return Response(
    'Could not verify your access level for that URL.\n'
    'You have to login with proper credentials', 401,
    {'WWW-Authenticate': 'Basic realm=""'})

def requires_auth(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        auth = request.authorization
        if not auth:
            return authenticate()
        api = check_auth(auth.username, auth.password)
        return f(api, *args, **kwargs)
    return decorated

@app.route('/')
def index():
    if request.authorization:
        return render_template("index.html")
    else:
    	# that's where your login form is
        return render_template("login.html")

@app.route('/login')
@requires_auth
def login(api):
    if api:
        return jsonify(), 200
    else:
        abort(400)
```

And one more thing, [here is](https://cscalendar.herokuapp.com) how it looks like.

