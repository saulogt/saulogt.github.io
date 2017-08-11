---
layout: post
title:  "How I solved a memory leak in a NodeJS worker"
date:   2017-09-09-20 09:00:00
categories: []
tags: [ NodeJS, Q, Promise, Events]
#image: "/images/2016/11/UIKevent/controls.png"
comments: true
published: false
---

My first challenge in my new job was to solve a memory leak in a NodeJS worker on Heroku that existed for about 2 year. Back then it was not an issue since Heroku restarts the server every 24 hour and the volume of users was still low. The problem is that now it was crashing sometimes twice a day and affecting performance.

I haven't done this kind of investigation on a NodeJS app, but I've done that many times on C and C++, so I had an idea what to expect. In fact it was much easier than I thought.

This is how it was working after I deployed the solution:
![Result 1](/images/2017/08/memoryleak-sol1.png)

And this is some days later. Beautiful don't you think?
![Result 2](/images/2017/08/memoryleak-sol2.png)