---
layout: post
date: 2024-02-07 12:50:00
title: "My new side project - Optogrid"
tags: [Postgres, Next.JS]
# image: "/images/2020/10/cookies.jpg"
published: true
comments: true
---

Since I was a kid, my mom has been working in the Optical industry, and when I was about 10 years old she opened her own small business. I've always heard her complaining about the market, the difficulties of being the owner and so on. I promised myself that I would do something else for a living and I fell in love with computer programming very early in my life. However, I learned a lot by osmosis 😂. And recent I decided to help my parents and my brother (Yes, my brother when on the same track) by building a tool to allow them to sell prescription eyeglasses remotely. And maybe serve someone else with the same use case

So I created Optogrid [optogrid.io](https://www.optogrid.io?utm_source=saulo_blog&utm_medium=link&utm_campaign=side_project).

The tool is used to measure PD (pupillary distance), Dual PD and Segment Height.

It took to long to release, because every time I found a new cool thing to integrate there.

Here is the stack:

- Next.JS
- Postgres (Vercel)
- Firebase (Only auth)
- pnpm (Monorepo is great)
- Sentry (I used to use Rollbar, but decided to try something else)

The app is currently free, but I plan to add a paywall soon.
Link directly to the app [app.optogrid.io](https://app.optogrid.io?utm_source=saulo_blog&utm_medium=link&utm_campaign=side_project_app)
