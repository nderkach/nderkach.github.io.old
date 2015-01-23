---
layout: post
title: "How to setup a typical Python development environment on openSUSE"
categories: [opensuse, python, heroku, zsh, virtualenv, pip]
description: "How to setup a typical Python development environment on openSUSE"
comments: true
---

It's been a long time since I've last installed a Linux distro on my desktop, more than 3 years I'd say. Naturally I was quiet excited to watch the breakthrough it made over that time. Since my previous macbook air kicked the calendar, I figured time has come to give openSUSE a spin.

Frankly I was quiet impressed that everything went pretty smooth, apart from the fact that I had to configure my BIOS to disable UEFI Secure Mode and that double-finger scrolling didn't work on my touchpad out of the box (yikes!)

I wanted to build a python app using Flask as a web-framework and I'm using Heroku for deployment. Here is how I went ahead an setup everything required to get started.
Ah yes, I'm using python3 and so should you.

<!--more-->

I've installed python and pip with zypper (`sudo zypper install`)

* python3
* python3-pip

I love using zsh, because it's [awesome](http://fendrich.se/blog/2012/09/28/no)

 {% highlight bash %}
 install zsh
 sudo zypper install zsh
 # make zsh your login shell
 chsh -s $(which zsh)
 # relogin to make it work
 {% endhighlight %}

Then I need to setup my [heroku toolbelt](https://toolbelt.heroku.com/standalone):

{% highlight bash %}
wget -qO- https://toolbelt.heroku.com/install.sh | sh
{% endhighlight %}

Adjust your path to use heroku tools, but instead of adjusting ~/.profile use ~/.zshrc or ~/.zshenv

go ahead and setup your heroku tools as described in the guide, but don't forget to install Ruby first. Also, don't forget to add your ssh key in your heroku account settings.

Here is the catch though: for [some reason](https://help.heroku.com/tickets/101227) foreman (used for local debugging) wasn't included in the bundle, so you have to run

{% highlight bash %}
sudo gem install foreman
foreman1.9 start
{% endhighlight %}

to start it.

And yes, be a good chap and install [virtualenv](http://docs.python-guide.org/en/latest/dev/virtualenvs/) as well:

{% highlight bash %}
sudo pip-3.3 install virtualenv
# cd to you app
virtualenv-3.3 venv --distribute
source venv/bin/activate
{% endhighlight %}

Now, HACK.

