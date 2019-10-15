---
title: Handling Cross-Origin Resource Sharing (CORS) Request
description: Handling CORS request in Fano Framework
---

<h1 class="major">Handling CORS</h1>

## What is CORS request?

Cross-Origin Resource Sharing (CORS) request is HTTP request which `Origin` header
does not same with `Host` header. By default browser blocks such requests due to same origin policy.

This may pose some problem for backend API, which sometime, it is configured to run
on different host or port.

To allow such requests, backend API needs to tells browser that it acknowledges such request
by sending appropriate response header.

## Handling CORS with middleware

Fano Framework provides built-in middleware class `TCorsMiddleware` which is to simplify task for sending appropriate CORS headers. Read [Middlewares](/middlewares) for more information about working with middlewares.

Constructor of `TCorsMiddleware` expects `ICors` interface instance which responsible to handle CORS request.

`TNullCors` is `ICors` implementation which simply allow all CORS request without restriction, while `TCors` class is implementation which can be configured to selectively apply restriction.

## Register CORS middleware with container

Fano Framework has `TCorsMiddlewareFactory` or `TNullCorsMiddlewareFactory` class
which allows you to register `TCorsMiddleware` with `TCors` or `TNullCors` with service container.
Both factory classes are derived from `TBaseCorsMiddlewareFactory`.

```
container.add('globalMiddlewares', TMiddlewareListFactory.create());
container.add(
    'cors',
    (TCorsMiddlewareFactory.create())
        .allowedOriginsPatterns(['http\:\/\/localhost\:[0-9]{1,5}'])
        .allowedMethods(['GET', 'POST', 'OPTIONS'])
        .allowedHeaders(['X-My-Custom-Header'])
);
```

To use `TNullCorsMiddlewareFactory`, just replace `TCorsMiddlewareFactory` above.

## Register dispatcher with support middleware

Because we need to execute middlewares, we cannot use `TSimpleDispatcher` class which
by default is already registered when we use, for example, `TSimpleWebApplication` class.

So we need to use `TDispatcher` class which support middlewares.

```
var globalMiddlewares : IMiddlewareLinkList;
...
globalMiddlewares := container.get('globalMiddlewares') as IMiddlewareLinkList;
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

Attach CORS middleware instance to application middleware collection to ensure
CORS middleware is executed for all application routes.

```
globalMiddlewares.add(container.get('cors') as IMiddleware);
```

## Configure CORS settings

`TBaseCorsMiddlewareFactory` class provides several methods to help configure CORS
settings.

### Allowed origins

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

### Allowed origins patterns

You can also configure origin to allow using regex pattern. Following code will cause
any request coming from `http://localhost:port` where port is integer value.

```
factory.allowedOriginsPatterns(['http\:\/\/localhost\:[0-9]{1,5}']);
```

So any requests from `http://localhost:9000` or `http://localhost:9100` are allowed but not
`http://127.0.0.1:9000`.

### Allowed methods

To allowed only HTTP GET and POST

```
factory.allowedMethods(['GET', 'POST']);
```

To allowed all method

```
factory.allowedMethods(['*']);
```

### Allowed headers

To allowed custom HTTP header

```
factory..allowedHeaders(['X-My-Custom-Header']);
```

To allowed all custom headers

```
factory.allowedHeaders(['*']);
```

### Exposed headers

To list HTTP headers exposed by application that browser allowed to access.

```
factory.exposedHeaders(['X-My-Custom-Header']);
```

To expose all headers

```
factory.exposedHeaders(['*']);
```

### Credentialed request

To allow credentialed request which aware of HTTP Cookies and HTTP Authentication.

```
factory.supportCredentials(true);
```

### Max age

To set number of seconds, preflight request can be cached by browser, set it as follows

```
factory.maxAge(3600);
```

## Testing CORS feature

Create a backend application that handle CORS, for example, you can use [Fano Cors example application](https://github.com/fanoframework/fano-cors) as base.

Create a simple web page to call our API via ajax, for example

```
<html>
<head><title>CORS Test</title></head>
<body>
    <div id="content"></div>
    <button id="btnLoad">Load</button>
    <script src="https://code.jquery.com/jquery-3.4.1.min.js"></script>
    <script>
        $(document).ready(function(){
            $('#btnLoad').click(function(){
                $.ajax({
                    url: "http://fano-cors.fano/",
                    headers: { 'x-my-custom-header': 'some value' }
                }).then(function(resp){
                    $('#content').text(resp.hello);
                });
            });
        });
    </script>
</body>
</html>
```
Save code above as `client.html` in your web server document root. It is assumed that our application is at `http://fano-cors.fano/`.

Open browser and go to `http://localhost/client.html`, click `Load` button
to execute ajax request to our API.

Browser will send ajax request with header `Origin` equals to `http://localhost/client.html` and `Host` equals `http://fano-cors.fano/`, so request is a CORS request.

## Explore more

- [Middlewares](/middlewares)
- [Dispatcher](/dispatcher)
- [Fano Cors example application](https://github.com/fanoframework/fano-cors)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
