---
layout: post
title: "OpenSSL Is A Pain"
date: 2013-01-14 22:12
comments: true
categories: [Work, Code, OpenSSL]
---

*- Warning: Technical -*

Today I had the joy of dealing with a particularly annoying problem at work. What should have taken me about two minutes to set up, ended up taking an hour and a half and four people's time.

To understand this post, you first need to know a little bit about how some of [our software](http://kurogo.org/home/) works. Most of the modules in our software talk to external data sources, pulling in content and feeds from different types of backends the client may have. We have some modules however, that talk to our own internally developed backend. Our own backend we'll call the content server (CMS), and the consumer we'll call the application server (app server).

One of the things I needed to do today for one of our designers was to hook up the app server to the CMS, so he could theme the site and make it look good. This should have been a very simple process, but when I put everything in place I got a lovely error saying the CMS wasn't available. I was able to make the calls via my browser, so I knew the CMS was functioning fine, must be an issue with the app server. I added some debugging code to the component that makes the request, and was greeted by a lovely cURL SSL error:

```
curl: (35) error:14077458:SSL routines:SSL23_GET_SERVER_HELLO:reason(1112)
```

I did what all good software engineers do, and Googled the error, which lead me to [this ticket](http://sourceforge.net/p/curl/bugs/1037/). The short of it is that when a client using OpenSSL 0.9.8 connects to a server using OpenSSL 1.0, a TLS alert is raised, which causes OpenSSL 0.9.8 to explode. The tricky part is that this only happens when the server doesn't have an Apache ServerName or ServerAlias option that matches the request domain name. It just so happened that our CMS server was running 1.0, and my local machine was running 0.9.8. The CMS also didn't have a valid ServerName set, and so I got this error. We have several other instances of this type of connection (app server to CMS) running, but this was the first time it happened in the particular manner.

We found a couple fixes. The easiest is to simply add a matching ServerName option to the CMS server. The other option was to force SSLv3 in cURL, which magically made this bug disappear. It was obnoxious to track down because no one else could reproduce it, but I was very clearly getting an error.

While I love OpenSSL and the wonders it offers, it can sometimes be very annoying to work with.