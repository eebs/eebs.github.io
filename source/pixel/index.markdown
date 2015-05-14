---
layout: page
title: "pixel"
date: 2015-05-14 12:39
comments: false
sharing: false
footer: true
---

Pixel is an IRC bot who tries to be helpful. Pixel is partially inspired by [Bucket](http://wiki.xkcd.com/irc/Bucket) who lives in the freenode.net #xkcd channel.

There are a number of different behaviors Pixel has, broken down below.

***

### YouTube

Anytime a YouTube link is posted in the channel, Pixel will post the title of the video in the channel.

***

### Team Funcom

Played on the Team Funcom Team Fortress 2 server recently? Check your list of awards or view how many players are currently playing. Use the following commands to get information about the TF2 server.

*   !tf2 - View the current map and number of players on the server.

*   !players - View a list of players currently playing.

*   !awards [name] - View a list of awards for yourself or [name] if provided. If no name is provided, Pixel will search for your IRC nick, and choose the first result.

***

### Facts

Pixel can be taught facts, and he will remember them forever!

#### Addressing Pixel

You need to address Pixel directly to teach him facts. You can do this by prefixing your message with "pixel:".

#### Teaching Facts to Pixel

##### X is Y

The most common type of fact you can teach to Pixel is by simply saying "X is Y". If someone says "X" later on, Pixel will reply "X is Y"

Be careful about accidentally creating a new fact while talking to Pixel.  
If you type "X is Y is Z", Pixel will split it on the first "is", making the trigger "X" and the factoid "Y is Z" and Pixel will respond "X is Y is Z".  
If you wanted 'X is Y' to trigger 'Z', teach Pixel 'X is Y \<is\> Z'. See \<verb\> below.

##### X \<verb\> Y

You can use other things instead of "is" by putting them inside greater and lesser-than signs, such as "X \<hates\> Y" or "X \<compliments\> Y"

##### X \<reply\> Y

You can say "X <\reply\> Y", when someone says "X", Pixel will respond "Y".

##### X \<action\> Y

If you use "X \<action\> Y", when someone says "X", Pixel will use "/me Y".

#### Username Variables

Putting a username variable in a factoid will replace the variable with a username when triggered.

##### $who

"$who" will be replaced by the name of the person who triggered the fact.

##### $someone

"$someone" will be replaced by a username chosen from anyone that Pixel has ever seen.

***

### Help

You are here.
