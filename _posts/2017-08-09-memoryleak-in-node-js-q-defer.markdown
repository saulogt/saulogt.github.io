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

My first challenge in my current job was to solve a memory leak in a NodeJS worker on Heroku that existed for about 2 years. Back then it was not a big deal since Heroku restarts the server every 24 hours and the volume of users was not relevant. The problem is that now it was crashing sometimes twice a day and affecting performance and the user experience.

I haven't done this kind of investigation on a NodeJS app before, but I've done that many times in C and C++, so I had an idea what to expect. In fact, it was much easier than I thought.


This is how it was working after I deployed the solution:
![Result 1](/images/2017/08/memoryleak-sol1.png)

And this is some days later. Beautiful don't you think?
![Result 2](/images/2017/08/memoryleak-sol2.png)

The first thing that I did was to take a heap snapshot using Chrome Dev Tools. Actually, as I was not familiar with the code, that was the only thing doable at that moment.

```
node --inspect app.js
Debugger listening on ws://127.0.0.1:9229/3e80809c-9ef8-4a2d-b402-5db73c30b7ed
For help see https://nodejs.org/en/docs/inspector
```

Chrome Dev Tools looks like this:

![Chrome Dev Tools](/images/2017/08/Developer_Tools_-_Node_js.png)

The secret is to take two or more snapshots letting the memory leak occur and compare the snapshots. And we don't need additional software for that.