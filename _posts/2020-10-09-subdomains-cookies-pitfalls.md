---
layout: post
date:   2020-10-09 09:00:00
title: "Cross-(sub)domains cookies pitfalls"
tags: [ NodeJs, cookies, React, cors, axios]
image: "/images/2020/10/cookies.jpg"
published: true
comments: true
---

This post is to describe the problem I had to configure a project to share cookies with all subdomains. It was supposed to be a simple task, but I spent many hours to figure out what I was doing wrong.

The project was the authentication for [coldletter.com](https://coldletter.com)


## My setup
I have a SPA running on `app.coldletter.com` that calls `/login` at `api.coldletter.com` using `axios`.

## First change

```js
authRouter.post(
  '/login',
  async (req, res) => {
    const loginRequest = req.body;
    const ret = await authService.login(loginRequest);

    res.cookie('sessionToken', ret.sessionId, {
        signed: true,
        secure: config.isProduction,
        maxAge: 1000 * 60 * 60 * 24 * 35,
        domain: 'coldletter.com',
    });

    res.status(200).send(ret);
  },
);

```

Great! After logging in, I noticed that the cookie is available in the browser of any subdomain. But... the cookie was never sent to the server.

After trying many different things, I realized that `axios` does not send cross-domain cookies by default. So I had to make the following change n the request in my SPA:

```js
return axios(`${API_URL()}/login`, {
    withCredentials: true, /// <<<< Here is the change !!!
    method: 'POST',
    data: {
      email,
      password,
    },
  })

```

Now, a new error happened...

## The new error

```
Access to XMLHttpRequest at '...' from origin '...' has been blocked by CORS policy: The value of the 'Access-Control-Allow-Origin' header in the response must not be the wildcard '*' when the request's credentials mode is 'include'. The credentials mode of requests initiated by the XMLHttpRequest is controlled by the withCredentials attribute.
```

Yeah, I was using `app.use(cors())` in this app.

Configuring CORS properly solved the problem.

```js
  app.use(
    cors({
      credentials: true,
      origin: [
          'https://coldletter.com',
          'https://api.coldletter.com',
          'https://app.coldletter.com',
      ],
    }),
  );
```