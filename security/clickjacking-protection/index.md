---
title: Clickjacking protection
description: Handling Clickjacking issue in Fano Framework
---

<h1 class="major">Clickjacking</h1>

## What is it?

[Clickjacking](https://owasp.org/www-community/attacks/Clickjacking) is a trick to make you click something attacker want while you intent to click top level page. It does it by adding invisible layer on top of legitimate content that you want to click. [Read clickjacking prevention from OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Clickjacking_Defense_Cheat_Sheet.html) for more information.

## Defending with X-Frame-Options header

`X-Frame-Options` response header tells browser whether or not it should allow to render a page in `<frame>` or `<iframe>`.

Fano Framework provides helper middleware, `TFrameGuardMiddleware` which adds this header to response. To use it, just add it to application global middleware list or a route.

For example, to deny render any `<iframe>` or `<frame>`.

```
router.get(
    '/',
    homeController
).add(TFrameGuardMiddleware.create('DENY'));
```

of if you want to use dependency container,

```
container.add('xssFilter', TXssFilterMiddlewareFactory.create().deny());
...
router.get(
    '/',
    homeController
).add(container['xssFilter'] as IMiddleware);
```

To allow only from same origin,

```
router.get(
    '/',
    homeController
).add(TFrameGuardMiddleware.create('SAMEORIGIN'));
```
or using factory
```
container.add(
    'xssFilter',
    TXssFilterMiddlewareFactory.create().sameOrigin()
);
```

To allow from certain domain,

```
router.get(
    '/',
    homeController
).add(TFrameGuardMiddleware.create('ALLOW-FROM http://example.com'));
```

```
container.add(
    'xssFilter',
    TXssFilterMiddlewareFactory.create().allowFrom('http://example.com')
);
```

- [Dispatcher](/dispatcher)
- [Security](/security)
- [Middlewares](/middlewares)
