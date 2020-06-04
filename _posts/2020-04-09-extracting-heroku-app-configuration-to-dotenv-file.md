---
layout: post
date:   2020-04-09 09:00:00
title: "Extracting Heroku app configuration to .env file"
tags: [ Heroku, NodeJs, tee, cli]
image: "/images/2020/04/heroku-logo.png"
published: true
comments: true
---

This is useful to debug your app using the same environment variable of your running Heroku app.<!-- more -->

## Requirements
- Heroku cli installed
- Heroku cli properly configured

Just run this in you project root (Or where the `.env` file is):

```bash
$ heroku config -a your-application-name -s | tee .env
```

The `.env` file will be overwritten with your Heroku app settings.

Now you can debug you app running on you workstation. 