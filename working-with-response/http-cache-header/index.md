---
title: Working with HTTP Cache Response Header
description: Tutorial on how to work with HTTP Cache Response header in Fano Framework
---

<h1 class="major">HTTP Cache Header</h1>

## Adding HttP Cache Header

To add http cache header ([RFC 7324](https://tools.ietf.org/html/rfc7234)), Fano Framework provides built-in [middleware](/middlewares)
`TCacheControlMiddleware`.

To add HTTP cache header, create middleware or register its factory class `TCacheControlMiddlewareFactory` within container.

```
container.add(
    'cacheControl',
    TCacheControlMiddlewareFactory.create().useETag()
);
```
and then, attach middleware to [route](/working-with-router) that will need to be added with HTTP cache header.

```
router.get(
    '/',
    container['homeController'] as IRequestHandler
).add(container['cacheControl'] as IMiddleware);
```
## How it works
When this middleware attached to route, additional steps is taken:

- If method is not cacheable (not GET nor HEAD) then nothing is done else,
- `Cache-Control` header is added to response header,
- If ETag is used then MD5 hash of response body is computed and `ETag` header is added to response with hash value of response body.
- If request contains `If-None-Match` header, its value is compared with ETag value, if they are matched or `If-Non-Match` equals `"*"`, then response is considered not modified.
- If response contains header `Last-Modified` and request contains header `If-Modified-Since`, both headers values are compared. If `Last-Modified` is older then response is considered not modified.
- If response is not modified, instance `TNotModifiedResponse` is returned instead. Read [Not modified response](/working-with-response#not-modified-response) section for more information this class.

## Setting Cache-Control header value

### Set type
`cacheType()` method set type of cache, accepts value of `ctPrivate` or `ctPublic` value. If it is not set `ctPrivate` is assumed.
```
container.add(
    'cacheControl',
    TCacheControlMiddlewareFactory.create().cacheType(ctPublic)
);
```

### Prevent cache to be stored
`noStore()` method enable `no-store` value.
```
container.add(
    'cacheControl',
    TCacheControlMiddlewareFactory.create().noStore()
);
```
### Set max-age value
`maxAge()` method set `max-age` value.
```
container.add(
    'cacheControl',
    //set max-age=3600 (1 hour)
    TCacheControlMiddlewareFactory.create().maxAge(60*60)
);
```
if max-age is set with 0 value, it implies `no-cache`.

### Enable ETag header
`useETag()` method enable `ETag` header. When enable, `ETag` response header is added to response with value equals to MD5 hash of response body.

```
container.add(
    'cacheControl',
    TCacheControlMiddlewareFactory.create().useETag()
);
```
### Set must-revalidate
`mustRevalidate` method enable `must-revalidate` value.

```
container.add(
    'cacheControl',
    TCacheControlMiddlewareFactory.create().mustRevalidate()
);
```

All methods above can be chained,
```
container.add(
    'cacheControl',
    TCacheControlMiddlewareFactory.create()
        .cacheType(ctPublic)
        .mustRevalidate()
        .useETag()
);
```

## Explore more

- [Working with Request](/working-with-request)
- [Working with Response](/working-with-response)
- [Middlewares](/middlewares)
- [Fano Cache Control example](https://github.com/fanoframework/fano-cache-control)
