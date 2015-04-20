---
layout: post
title: First impressions about the AWS Elastic Beanstalk
date: "2015-04-08 22:16"
categories: []
tags: [aws ,elastic beanstalk ,nodejs]
published: true
---

This is my first experience configuring an web application environment on AWS from acquisition to deployment. As I'll probably make many mistakes, I decided to write this post as a ship's log, in order to register my impressions, problems and doubts during the process.  

## Just registered to AWS, what now?

Well... Now I have a Free tier for one year and that gives me:
- 750 hours per month of a micro EC2 instance running Linux
- 750 hours per month of a micro EC2 instance running Windows
- 5 GB of S3 storage
- A bunch of other things that I didn't assume that was important. (baybe I'm wrong)

![AWS Options](/images/2015/04/aws_options.png)

I got this screen above and felt sad about the plenty of options that I have now. [The paradox of choice](http://www.ted.com/talks/barry_schwartz_on_the_paradox_of_choice?language=en)

At this point, I didn't search for any tutorial on the Web and stuck with the official content, as I was afraid of paying more than I have in Free tier (that is zero $). I understood that I should create a EC2 instance to see things working.

Then I watched this and all my problems were resolved:
<iframe width="560" height="315" src="https://www.youtube.com/embed/dvmssHHBnII?rel=0" frameborder="0" allowfullscreen="allowfullscreen">
</iframe>

I created a node application with a mouse click, waited a couple of minutes and I had a sample application running. It created all the instances and configuration needed to run an environment like that.
Too good to be true... Now I don't know how to deploy a new app, and every try ended up with 502 bad gateway. Yes, I know that I should read the documentation first, but my anxiety don't let me do that now.

Until now I spent about 2 hours.

Next step: Deploy successfully a nodejs application.

Edit 2015-04-13:

I have just discovered that I created my environment in the wrong region. As my goal is to play a little with the new Amazon Machine Learning, I had to delete my environment and recreate it in the region Virginia.
