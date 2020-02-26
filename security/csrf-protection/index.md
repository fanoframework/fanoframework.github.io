---
title: Protecting against Cross-Site Request Forgery (CSRF) attack
description: Handling Cross-Site Request Forgery (CSRF) issue in Fano Framework
---

<h1 class="major">Cross-Site Request Forgery (CSRF)</h1>

## What is CSRF?

According to [OWASP](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF))

> Cross-Site Request Forgery (CSRF) is an attack that forces an end user to execute unwanted actions on a web application in which they're currently authenticated. CSRF attacks specifically target state-changing requests, not theft of data, since the attacker has no way to see the response to the forged request. With a little help of social engineering (such as sending a link via email or chat), an attacker may trick the users of a web application into executing actions of the attacker's choosing. If the victim is a normal user, a successful CSRF attack can force the user to perform state changing requests like transferring funds, changing their email address, and so forth. If the victim is an administrative account, CSRF can compromise the entire web application.

## Protecting against CSRF with middleware

Fano Framework provides built-in middleware class `TCsrfMiddleware` which is to simplify task for protecting against CSRF attack. Read [Middlewares](/middlewares) for more information about working with middlewares.

Constructor of `TCsrfMiddleware` expects 5 parameters,

- `ICsrf` instance which responsible to generate CSRF token and also check token validity,
- `ISessionManager` interface instance which responsible to maintain CSRF token between request,
- `IRequestHandler` interface instance responsible to generate response when CSRF token check is failed.
- Optional name of field for CSRF name, if not provided `crsf_name` is assumed,
- Optional name of field for CSRF token, if not provided `crsf_token` is assumed.

Read [Working with Session](/working-with-session) for more information about `ISessionManager`.

## Register global middleware list

```
container.add('globalMiddlewares', TMiddlewareListFactory.create());
globalMiddlewares := container.get('globalMiddlewares') as IMiddlewareList;
```

## Register CSRF middleware with container

Fano Framework has `TCsrfMiddlewareFactory` class which allows you to register `TCsrfMiddleware` service container.

```
container.add(
    'verifyCsrfToken',
    TCsrfMiddlewareFactory.create(config.getString('secretKey'))
);
```

## Register dispatcher with support middleware

We need to use dispatcher class which support middlewares and sessions.

```
container.add(
    GuidToString(IDispatcher),
    TSessionDispatcherFactory.create(
        globalMiddlewares as IMiddlewareLinkList,
        container.get(GuidToString(IRouteMatcher)) as IRouteMatcher,
        TRequestResponseFactory.create(),
        container.get(GuidToString(ISessionManager)) as ISessionManager,
        (TCookieFactory.create()).domain(config.getString('cookie.domain')),
        config.getInt('cookie.maxAge')
    )
);
```

Read [Dispatcher](/dispatcher) and [Dependency Container](/dependency-container) for more information.

## Attach CSRF middleware to application middleware

Attach CSRF middleware instance to application middleware collection to ensure
CSRF middleware is executed for all application routes.

```
globalMiddlewares.add(container.get('verifyCsrfToken') as IMiddleware);
```

## Get current CSRF token

When you attach `TCsrfMiddleware` middleware to application middleware list, everytime request is coming, new random token and name are generated and it can be read from current session variable.

```
function THomeController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
var sess : ISession;
begin
    sess := fSessionManager.getSession(request);
    viewParams.setVar('csrfName', sess.getVar('csrf_name'));
    viewParams.setVar('csrfToken', sess.getVar('csrf_token'));
    result := inherited handleRequest(request, response, args);
end;
```

By default, name and token field is `csrf_name` and `csrf_token` respectively. When you build your HTML form, you need to ensure correct name is used,

```
<form method="post" action="/">
    <input type="hidden" name="csrf_name" value="[[csrfName]]">
    <input type="hidden" name="csrf_token" value="[[csrfToken]]">
...
</form>
```

## Verify CSRF token

CSRF token verification is done in CSRF middleware automatically for POST, PUT, DELETE and PATCH request by comparing `csrf_name` and `csrf_token` values in request against corresponding values stored in session.

If they are matched, execution continues to next middlewares otherwise it stops by calling failure request handler you set when creating CSRF middleware.

CSRF token is for one-time use only. After token verifiction, new token and name is generated and then it is replaced old token and name in session.

## Configure CSRF middleware settings

`TCsrfMiddlewareFactory` class provides several methods to help configure CSRF middleware.

### Change name and token field

To change name and token field

```
var factory : IDependencyFactory;
...
factory := TCsrfMiddlewareFactory.create()
    .nameField('my_cool_name')
    .tokenField('my_cool_token');
```

You need to ensure correct name is used in HTML form and in code that read token

```
viewParams.setVar('csrfName', sess.getVar('my_cool_name'));
viewParams.setVar('csrfToken', sess.getVar('my_cool_token'));
```

```
<form method="post" action="/">
    <input type="hidden" name="my_cool_name" value="[[csrfName]]">
    <input type="hidden" name="my_cool_token" value="[[csrfToken]]">
...
</form>
```

### Change token generator

`TCsrfMiddlewareFactory` class, by default, uses `TCsrf` class to generate random token and to verify token. `TCsrf` uses `createGUID()` and SHA1 hash function to generate random token which may not be adequate. If you need stronger token generator, you can replace with your own implementation by creating class which implements `ICsrf` interface and provides following methods

```
function generateToken(out tokenName : string; out tokenValue : string) : ICsrf;

function hasValidToken(
    const request : IRequest;
    const sess : ISession;
    const nameKey : shortstring;
    const valueKey : shortstring
) : boolean;
```

and then replace token generator

```
factory := TCsrfMiddlewareFactory.create(config.getString('secretKey'))
    .nameField('my_cool_name')
    .tokenField('my_cool_token')
    .csrf(TMyOwnCsrf.create());
```

### Change failure handler

By default, when CSRF token check is failed, it calls instance of `TDefaultFailCsrfHandler` class which returns response with error code HTTP 400 (Bad Request) and message 'Fail CSRF check'.

You may want to replace it with your own request handler by calling `failureHandler()` method.

```
factory := TCsrfMiddlewareFactory.create()
    .nameField('my_cool_name')
    .tokenField('my_cool_token')
    .failureHandler(TMyOwnCsrfFailController.create());
```
Failure handler class must implements `IRequestHandler` interface.

### Change session manager

Csrf middleware needs to get session from current request, thus requires access to `ISessionManager` interface instance. By default if not set, factory class tries to get session manager by requesting service container as down in following code

```
if fSessionManager = nil then
begin
    fSessionManager := container.get(GuidToString(ISessionManager)) as ISessionManager;
end;
```
You can set it manually if required,

```
factory := TCsrfMiddlewareFactory.create()
    .sessionHandler(container.get('sessMgr') as ISessionManager);
```
## Explore more

- [Middlewares](/middlewares)
- [Dispatcher](/dispatcher)
- [Working with Session](/working-with-session)
- [Fano Csrf example application](https://github.com/fanoframework/fano-csrf)
