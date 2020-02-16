---
title: HTTP Authentication
description: Handling HTTP authentication in Fano Framework
---

<h1 class="major">HTTP Authentication</h1>

## Basic Authentication

Besides usual web application form-based authentication, developer can use authentication mechanism that is provided by HTTP protocol as described in [RFC 7235](https://tools.ietf.org/html/rfc7235).

Currently, Fano Framework supports `Basic` type HTTP authentication only.

## Handling Basic Authentication with middleware

Fano Framework provides implementation for basic type HTTP authentication with
`TBasicAuthMiddleware` middleware with purpose to simplify task to protect access of certain routes using HTTP authentication.

Read [Middlewares](/middlewares) for more information about working with middlewares.

Constructor of `TBasicAuthMiddleware` expects two parameters,

- Instance of `IAuth` interface which is responsible to perform actual authentication, for example, reading database for existence of credential being check.
- String of realm name.

`TStaticCredentialsAuth` is built-in `IAuth` implementation which test credential against array of allowed credentials.

Developer can create their own custom authentication by implementing `IAuth` interface and inject it to `TBasicAuthMiddleware` constructor.

## How Basic Auth middleware works

Everytime a user tries to access route that is protected with `TBasicAuthMiddleware` middleware, user Internet browser, initially does not know that this route requires authentication, thus will cause our application to response with HTTP 401 error code and header `WWW-Authenticate` set with value of `Basic realm="[realm name]"`.

Upon receiving this response, browser will prompt user to provide their credential. The supplied credential are then sent back to our application. If credential is valid, our middleware continues to execute next middleware and eventually controller. If credential is not valid, our middleware returns same HTTP 401 response.

## Register Basic Authentication middleware with container

Fano Framework has `TStaticCredentialsBasicAuthMiddlewareFactory` class
which allows you to register `TBasicAuthMiddleware` using `TStaticCredentialsAuth` into service container.

```
container.add(
    'basicAuthMiddleware',
    TStaticCredentialsBasicAuthMiddlewareFactory.create('fano-realm')
        .addCredential('hello@fano', 'world')
        .addCredential('hi@fano', 'nice')
);
```

Above code simplifies and does same thing as following code

```
var authMw : IMiddleware;
    allowedCredentials : TCredentials;
...
setlength(allowedCredentials, 2);
allowedCredentials[0].username = 'hello@fano';
allowedCredentials[0].password = 'world';
allowedCredentials[1].username = 'hi@fano';
allowedCredentials[1].password = 'nice';

authMw := TBasicAuthMiddleware.create(
    TStaticCredentialsAuth.create(allowedCredentials),
    'fano-realm'
);

```

Where `TCredentials` is declared as follows,

```
type

    TCredential = record
        username : string;
        password : string;
    end;

    TCredentials = array of TCredential;

```
## Register dispatcher with support middleware

You need to use `TDispatcher` class which support middlewares.

```
container.add('appMiddlewares', TMiddlewareListFactory.create());

container.add(
    GuidToString(IDispatcher),
    TDispatcherFactory.create(
        container['appMiddlewares'] as IMiddlewareLinkList,
        routeMatcher,
        TRequestResponseFactory.create()
    )
);
```

## Attach Basic Auth middleware to application routes

Attach basic auth middleware instance to application routes, for example

```
router.get('/', container['homeController'] as IRequestHandler)
    .add(container['basicAuthMiddleware'] as IMiddleware);
```

## Security consideration

Please note that because username and password is transmitted as Base64-encoded string, it is prone to man in middle attack. It is must be used in conjunction with SSL/TLS.

If your application is running behind reverse proxy, for example, with `mod_proxy_scgi` module, Apache does not pass `Authorization` header to application because of security concern.

In case, you have trusted network between Apache and your application, simple solution is to transform `Authorization` header into `HTTP_AUTHORIZATION` environment variable using `mod_rewrite` module.

```
<IfModule mod_rewrite.c>
    RewriteEngine on
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization},L]
</IfModule>
```

## Explore more

- [Middlewares](/middlewares)
- [Dispatcher](/dispatcher)
- [Fano Basic Auth example application](https://github.com/fanoframework/fano-basic-auth)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
