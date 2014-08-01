---
layout: post
title: "Up and Running with Octopress"
date: 2013-01-14 17:52
comments: true
categories: [Octopress, Ruby, Github]
---

As you can clearly see, I just finished installing [Octopress](http://octopress.org/). Over the past few weeks I have had sudden urges to write things down, either for discussion or mental relief. I used to have a blog on Wordpress several years ago, but I stopped writing for a while, and eventually trashed it. I figured I'd start one up again, using it to document various projects or ideas of mine.

I saw a post on Reddit about installing Octopress on Windows, and something about the Octo/Github wording made me click the link. Once I figured out what Octopress was, I realized I most definitely wouldn't want to install it on Windows, but I may check it out on my laptop. Octopress is a "blogging framework for hackers", which sounds fairly intriguing. It's built to use [Jekyll](https://github.com/mojombo/jekyll), something I had heard of in the past, mostly from [Kyle Rush's](http://kylerush.net/blog/meet-the-obama-campaigns-250-million-fundraising-platform/) blog post on the Obama fundraising platform. It's written in Ruby, my new favorite language, so I decided I'd try it out.

I followed the installation instructions on the Octopress page, and the install went smoothly. Github pages took a little while to build, but once it was up it worked well. I even managed to hook it up to my own domain. Using git to make posts is a little weird, but I think I'll get used to the process. There are some helper commands (implemented as rake tasks) that you can run to make new posts, generate the site, and even deploy, which makes things much easier. I'm still going to have to write some aliases because for whatever reason I have to run `bundle exec rake` instead of just `rake`. A couple of the commands seem to be often run in sequence, so I'll probably make an alias to run them together. I'll try and remember to post those here somewhere (I think Octopress has built in gist support or something).

*- Edit -* Found the gist support:

{% gist 4535615 %}

I'm not entirely sure what I'll end up using this blog for mostly, but it'll probably be a fair amount of everything. Maybe there will be some rants about code, or links to neat articles. I might add some posts about EvE Online, or maybe some entries about my ongoing Rails projects.

Who knows.
