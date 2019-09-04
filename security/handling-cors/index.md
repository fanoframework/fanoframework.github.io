---
title: Handling Cross-Origin Resource Sharing (CORS) Request
description: Handling CORS request in Fano Framework
---

<h1 class="major">Handling CORS</h1>

## What is CORS request?

Cross-Origin Resource Sharing (CORS) request is HTTP simple request which `Origin` header
does not same with `Host` header. By default browser blocks such requests.

This may pose some problem for backend API where sometime it is configure to run
on different host or port.

To allow such requests, backend API needs to tells browser that it acknoledges such request
by sending appropriate response header.

## Handling CORS with middleware

Fano Framework provides built-in middleware class `TCorsMiddleware` or
`TNullCorsMiddleware` which simplify task for sending appropriate CORS headers.
`TNullCorsMiddleware` is CORS middleware implementation which simply allow all CORS request without restriction, while `TCorsMiddleware` class is implementation which can be configured to selectively apply restriction.

## Register CORS middleware with container

Fano Framework has `TCorsMiddlewareFactory` or `TNullCorsMiddlewareFactory` class
which allows you to register `TCorsMiddleware` or `TNullCorsMiddleware` with service container.
Both factory classes are derived from `TBaseCorsMiddlewareFactory`.

```
container.add('globalMiddlewares', TMiddlewareCollectionAwareFactory.create());
container.add(
    'cors',
    (TCorsMiddlewareFactory.create())
        .allowedOriginsPatterns(['http\:\/\/localhost\:[0-9]{1,5}'])
        .allowedMethods(['GET', 'POST', 'OPTIONS'])
        .allowedHeaders(['X-My-Custom-Header'])
);
```

## Register dispatcher with support middleware

```
var globalMiddlewares : IMiddlewareCollectionAware;
...
globalMiddlewares := container.get('globalMiddlewares') as IMiddlewareCollectionAware;
container.add(
    GuidToString(IDispatcher),
    TDispatcherFactory.create(
        globalMiddlewares,
        container.get(GuidToString(IRouteMatcher)) as IRouteMatcher,
        TRequestResponseFactory.create()
    )
);
```

## Attach CORS middleware to application middleware

To attach CORS middleware instance to application middleware collection

```
globalMiddlewares.addBefore(container.get('cors') as IMiddleware);
```

## Configure CORS settings

`TBaseCorsMiddlewareFactory` class provides several methods to help configure CORS
settings.

## Allowed origins

To allow request from `http://cors.fano` and `http://my.fano`.

```
var factory : TBaseCorsMiddlewareFactory;
...
factory := TCorsMiddlewareFactory.create();
factory.allowedOrigins([ 'http://cors.fano', 'http://my.fano']);
```

To allow request from any origin

```
factory.allowedOrigins(['*']);
```

## Allowed origins patterns

You can also configure origin to allow using regex pattern. Following code will cause
any request coming from `http://localhost:port` where port is integer value.

```
factory.allowedOriginsPatterns(['http\:\/\/localhost\:[0-9]{1,5}']);
```

So any requests from `http://localhost:9000` or `http://localhost:9100` are allowed but not
`http://127.0.0.1:9000`.

## Allowed methods

To allowed only HTTP GET and POST

```
factory.allowedMethods(['GET', 'POST']);
```

To allowed all method

```
factory.allowedMethods(['*']);
```

## Allowed headers

To allowed custom HTTP header

```
factory..allowedHeaders(['X-My-Custom-Header']);
```

To allowed all custom headers

```
factory.allowedHeaders(['*']);
```

## Exposed headers

To list HTTP headers exposed by application that browser allowed to access.

```
factory.exposedHeaders(['X-My-Custom-Header']);
```

To expose all headers

```
factory.exposedHeaders(['*']);
```

## Credentialed request

To allow credentialed request which aware of HTTP Cookies and HTTP Authentication.

```
factory.supportCredentials(true);
```

## Max age

To set number of seconds, preflight request can be cached by browser, set it as follows

```
factory.maxAge(3600);
```

## Explore more

- [Middlewares](/middlewares)
- [Dispatcher](/dispatcher)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
