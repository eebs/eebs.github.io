---
layout: post
title: "Nginx and Unsupported HTTP Headers"
date: 2013-01-21 17:44
comments: true
categories: [Ruby, Rails, Nginx, Code]
---

*- Warning: Technical -*

One of my favorite pastimes is combining coding and video games. This weekend was full of coding, cursing, and hair pulling (some of the cursing was due to watching the Patriots lose). I play the video game [EvE Online](http://www.eveonline.com) and one of the fantastic things CCP, the company that makes EvE, has done is to provide an API into the game universe. This means you can make calls to the API to find out information about your characters, their skills, tasks they are performing in-game, etc. In EvE, players fly spaceships throughout the universe, fighting each other, NPCs, and in general blowing up lots of things. The ships players fly don't appear out of nowhere however, they must be built by other players in the game. That's where a friend of mine and I come in. We build capital ships, some of the largest ships in the game. Each ship takes many takes to build, and there is a somewhat lengthy process we go through in game. I have created [a website](http://ships.eebsy.com/), that helps us track the progress and various stages of production of all our ships. It is mostly for our internal use, but I have grand schemes for its eventual capabilities. It this website that this post is about.

<!-- more -->

Another cool part of EvE Online, is that CCP built a web browser right into the game client. It functions like a normal browser for the most part, but people (like myself) can build websites to take advantage of some extra features it offers. If a player uses the in game browser (IGB for short) to visit a site, and gives that site their trust, CCP sends to the site some extra information about the character that is visiting the site. This information contains things like the character's name, its ID, the corporation it belongs to, where it is, what ship it's flying, the list goes on. I decided to use this information to allow users to sign up for my site. When a character uses the IGB to sign up, my site takes the information from CCP and automatically fills in their character name, and only asks the user for a password. This ensures that only the person who owns the character can sign up. There are ways around this, it's not 100% foolproof, but it makes it harder to game the system.

So now you know my end goal. To use the information from the IGB to make user sign up easier. Now on to the cursing and hair pulling part. The way the IGB delivers this extra information, is through HTTP headers. The issue is that they are *invalid* HTTP headers. HTTP headers should be letters only, with hyphens separating words or phrases. *Accept*, *Cache-Control*, *Host*, and *User-Agent* are examples of some common headers you may have seen before. CCP however, sends headers like *EVE_TRUSTED*, *EVE_CHARNAME*, and *EVE_SHIPNAME*. These contain underscores, which are invalid HTTP header characters.

In most of my [Rails](http://rubyonrails.org/) applications, I use [nginx](http://wiki.nginx.org/Main) as my web server. It's very fast, uses very few resources, and is easy to configure. When I was first doing some testing, trying to get the extra header information from the IGB to my application, I noticed that nothing was coming through at all. All the headers that were supposed to be showing up simply weren't there. After some Googling, I quickly found out that the headers were invalid. There is even a mention of this [in the official wiki](http://wiki.eveonline.com/en/wiki/IGB_Headers#PHP), noting that if you're using nginx you should "activate the *underscores_in_headers* directive in your *server* config". So I did that. Unfortunately, it did nothing.

Here's what my nginx config looks like for this application:

```
upstream unicorn_ships {
  server unix:/tmp/unicorn.ships.sock fail_timeout=0;
}

server {
  listen 80;
  server_name ships.eebsy.com;
  root /home/deployer/apps/ships/current/public;

  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  try_files $uri/index.html $uri @unicorn;
  location @unicorn {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://unicorn_ships;
  }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 4G;
  keepalive_timeout 10;
}
```

I added the *underscores_in_headers* directive in the *server* block, right under the *root* directive. This is exactly where the EVE wiki says it should be, and also where the official documentation [says it should be](http://nginx.org/en/docs/http/ngx_http_core_module.html#underscores_in_headers). I messed around for about four hours on Sunday, trying to get the headers to go through. For a while, I thought the headers weren't being sent through the proxy, so I added a *proxy_set_header* entry to the *location* block, but that didn't help either.

Out of desperation, I added the *underscores_in_headers* entry to the main nginx.conf file. I typically don't like putting directives that only apply to one application in the root config file, but I didn't know what else to do. And of course, it starts working. Maybe I'm still too unfamiliar with configuring nginx, but the way I read the documentation, it makes it sound like it should work perfectly fine if you add it to the *server* block.

So after hours of cursing, debugging with cURL, and lots of frustration, my site finally works the way I want it to. It was definitely worth the effort, and I learned a few things along the way. If anyone knows why adding the directive to the *server* block didn't work, please let me know in the comments below.
