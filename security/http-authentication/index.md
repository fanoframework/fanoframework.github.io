---
title: HTTP Authentication
description: Handling HTTP authentication in Fano Framework
---

<h1 class="major">HTTP Authentication</h1>

## Basic Authentication

Besides usual web application form-based authentication, developer can use authentication mechanism that is provided by HTTP protocol as described in [RFC 2617](https://tools.ietf.org/html/rfc2617).

Currently, Fano Framework supports `Basic` and `Digest` HTTP authentication scheme only.

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

Upon receiving this response, browser will prompt user to provide their credential. The supplied credential are then sent back to our application thorough `Authorization` request header with value,

```
Authorization: Basic ["username:password" base64 encoded string]
```

For username `hello` and password `world`, Base64 encoded of string `hello:world` is `aGVsbG86d29ybGQ=`.

```
Authorization: Basic aGVsbG86d29ybGQ=
```

If credential is valid, our middleware continues to execute next middleware and eventually controller. If credential is not valid, our middleware returns same HTTP 401 response.

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

        //user-related data
        data : pointer;
    end;

    TCredentials = array of TCredential;

```
## Register dispatcher with middleware support.

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

Attach basic auth middleware instance to application [routes](/working-with-router), for example

```
router.get('/', container['homeController'] as IRequestHandler)
    .add(container['basicAuthMiddleware'] as IMiddleware);
```

## Handling Digest Authentication with middleware

Fano Framework provides implementation for digest type HTTP authentication with
`TDigestAuthMiddleware` middleware with purpose to simplify task to protect access of certain routes using Digest HTTP authentication scheme.

Constructor of `TDigestAuthMiddleware` expects three parameters,

- Instance of `IRandom` interface which is responsible to generate random value.
- Instance of `IAuth` interface which is responsible to perform actual authentication, for example, reading database for existence of credential being check.
- String of realm name.

`TDigestStaticCredentialsAuth` is built-in `IAuth` implementation which test credential against array of allowed credentials.

Developer can create their own custom authentication by implementing `IAuth` interface and inject it to `TDigestAuthMiddleware` constructor.

## Register Digest Authentication middleware with container

Fano Framework has `TStaticCredentialsDigestAuthMiddlewareFactory` class
which allows you to register `TDigestAuthMiddleware` using `TDigestStaticCredentialsAuth` into service container.

```
container.add(
    'digestAuthMiddleware',
    TStaticCredentialsDigestAuthMiddlewareFactory.create('fano-realm')
        .addCredential('hello@fano', 'world')
        .addCredential('hi@fano', 'nice')
);
```

## Attach Digest Auth middleware to application routes

Attach digest auth middleware instance to application routes, for example

```
router.get('/', container['homeController'] as IRequestHandler)
    .add(container['digestAuthMiddleware'] as IMiddleware);
```

## Handling Bearer Authentication with middleware

Fano Framework provides implementation for bearer type HTTP authentication with
`TBearerAuthMiddleware` middleware with purpose to simplify task to protect access of certain routes using Bearer HTTP authentication scheme.

Constructor of `TBearerAuthMiddleware` expects three parameters,

- Instance of `ITokenVerifier` interface which is responsible to perform actual token verification.
- String of realm name.
- String of credential key name where authenticated credential can be queried from request.

Currently, Fano Framework supports [JSON Web Token (JWT)](/security/jwt) verification only, via `TJwtTokenVerifier` class which implements `ITokenVerifier` interface.

After token is verified, credential info found in token is stored in request which later can be queried
in controller using name you defined in credential key name.

## Register Bearer Authentication middleware with container

Fano Framework has `TBearerAuthMiddlewareFactory` class
which allows you to register `TBearerAuthMiddleware` into service container.

```
container.add(
    'bearerAuth',
    TBearerAuthMiddlewareFactory.create()
        .realm('fano-realm')
        .verifier(container['tokenVerifier'] as ITokenVerifier)
        .credentialKey('fano_cred')
);
```

## Attach Bearer Auth middleware to application routes

Attach bearer auth middleware instance to application routes, for example

```
router.get('/', container['homeController'] as IRequestHandler)
    .add(container['bearerAuth'] as IMiddleware);
```
In code above, for each GET request to URL `/`, middleware will check token existence and verify
if it is found. If token is not found or not verified, it returns HTTP 401 response
with `WWW-Authenticate: Bearer realm="[realm name]"` header,
where `[realm name]` is realm that you set above.

When token is verified, credential found from token can be read from request in controller with
credential key you set before, i.e, `fano_cred`.

```
function THomeController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
var username : string;
begin
    username := request.getParam('fano_cred');
    response.body().write('Hello ' + username);
    result := response;
end;
```

## Security consideration

For Basic HTTP authentication scheme, username and password is transmitted as Base64-encoded string. It is prone to man in middle attack and it is must be used in conjunction with SSL/TLS.

Digest HTTP authentication scheme is more computation expensive but can be used with or without SSL/TLS because password and other data is sent to server as MD5 hashed value. However, because MD5 hash is not cryptographically strong, you need to be cautious when use it without SSL/TLS.

For Bearer HTTP authentication, token is credential. It must be used in conjunction with SSL/TLS
to avoid man in middle attack. Token may or may not be encrypted. If you use
unencrypted JWT token, do not store sensitive data in token.

To address when token is leaked to third party, You can combine short-lived token with short expiry time (access token) and login-lived token (refresh token). When access token is expired, client must get new access token from authentication server using refresh token. If refresh token is expired then client session is ended. To get new refresh token and access token, client must login again using username password.

### Missing Authorization header
If your application is running behind reverse proxy, for example, with `mod_proxy_scgi` module, Apache does not pass `Authorization` header to application because of security concern. This will cause all middlewares above return HTTP 401 as they can not find this header.

In case, you have trusted network between Apache and your application, simple solution is to transform `Authorization` header into `HTTP_AUTHORIZATION` environment variable using `mod_rewrite` module. You can add it in application virtual host configuration file.

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
