---
title: Cross-site Scripting (XSS) protection
description: Handling Cross-site scripting (XSS) issue in Fano Framework
---

<h1 class="major">Cross-site Scripting (XSS)</h1>

## What is it?

Cross-site scripting is a way for hacker to take over web page by injecting JavaScript into HTML document.

Basic prevention for this type of attack is to never trust input coming from user. If you must put data from user input into HTML document, sanitize it first. [Read XSS prevention from OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html) for more information.

## X-XSS-Protection header

While `X-XSS-Protection` header has been deprecated in modern browser. It is a quick win and basic protection against "Reflected XSS" attack but it does not prevent XSS attack in general.

Fano Framework provides helper middleware, `TXssFilterMiddleware` which adds this header to response. To use it, just add it to application global middleware list or a route.

```
router.get('/').add(TXssFilterMiddleware.create());
```

of if you want to use dependency container,

```
container.add('xssFilter', TXssFilterMiddlewareFactory.create());
...
router.get('/').add(container['xssFilter'] as IMiddleware);
```
## Explore more

- [Dispatcher](/dispatcher)
- [Security](/security)
- [Middlewares](/middlewares)
