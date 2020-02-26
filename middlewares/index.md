---
title: Middlewares
description: Documentation about how middlewares work in Fano Framework
---

<h1 class="major">Middlewares</h1>

## Why middleware?

In Fano Framework, middleware is an optional software component that is executed before or after request is passed to actual request handler. It is similar concept to firewall, in which, it can pass, block or modify [request](/working-with-request) or modify [response](/working-with-response).

For example, middleware allows developer to test if user is logged in before request reaches controller. If user is not logged in, it blocks request.
So when controller is executed, developer can be sure that user must be logged in.

Single middleware instance can be attach to one or more route. This allows centralized action to be taken for multiple controllers.

## Middleware architecture in Fano Framework

Fano Framework use simple chained middleware list. Each middleware can decide whether to pass request to next middleware or block. If middleware blocks a request, it must return response.

<a href="/assets/images/middlewares.svg">
<img src="/assets/images/middlewares.svg" alt="Middleware diagram" width="100%">
</a>

In Fano Framework, any class implements `IMiddleware` interface can be used as middleware. This interface has one methods `handleRequest()` which class must implements.

```
function handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader;
    const next : IRequestHandler
) : IResponse;
```
- `request` is current request object
- `response` is current response object
- `args` is current route arguments. Read [Working with Router](/working-with-router#getting-route-argument) for more information about route argument.
- `next` is next middleware to execute

If a middleware should let execution to continue, it must call `next` request handler otherwise execution is stop and response that is returned by middleware's `handleRequest()` method will be response what client browser received.

For example, following code will stop execution if request is not AJAX request otherwise it will continue execution.

```
function TAjaxOnlyMiddleware.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader;
    const next : IRequestHandler
) : IResponse;
begin
    if (not request.isXhr()) then
    begin
        response.headers().setHeader('Status', '403 Not Ajax Request');
        result := response;
    end else
    begin
        result := next.handleRequest(request, response, args);
    end;
end;

```

## Type of middlewares

### Based on scope

#### Application middleware

Application middleware is middleware that is attached and applied globally to all routes.

#### Route middleware

Route middleware is middleware that is attached and applied to one or more specific routes.

### Based on order of execution

### Before middleware

Before middleware is any middleware that is executed *before* controller execution. This is mostly type of middleware that can be used modify request or act as gate that block or pass request.

```
function TMyMiddleware.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader;
    const next : IRequestHandler
) : IResponse;
begin
    doSomething();
    result := next.handleRequest(request, response, args);
end;

```

### After middleware

After middleware is any middleware that is executed *after* controller execution.

```
function TMyMiddleware.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader;
    const next : IRequestHandler
) : IResponse;
begin
    result := next.handleRequest(request, response, args);
    doSomething();
end;

```

## Creating middleware

```
var ajaxOnly : IMiddleware;
    authOnly : IMiddleware;
...
ajaxOnly := TAjaxOnlyMiddleware.create();
authOnly := TAuthOnlyMiddleware.create();
```

## Attaching middleware to application middleware

When you use initialize `IDispatcher` implementation which support middlewares, such as `TDispatcher` class, you are required to setup one global `IMiddlewareLinkList` instances which stores list of middlewares applied globally to all routes. Read [Dispatcher](/dispatcher) for more information.

As shown in following code

```
{-----------------------------------------------
  register middleware list for application
------------------------------------------------}
container.add('appMiddlewares', TMiddlewareListFactory.create());
```

In you dispatcher initialization, you need to set global middlewares collection to use by dispatcher instance.

```
{-----------------------------------------------
  setup basic application request dispatcher
  which support middleware
------------------------------------------------}
container.add(
    'dispatcher',
    TDispatcherFactory.create(
        container.get('appMiddlewares') as IMiddlewareLinkList,
        aRouterInst as IRouteMatcher
    )
);
```

and then you can register a middleware to global middlewares as follows

```
var appMiddlewares : IMiddlewareList;
...
appMiddlewares := container.get('appMiddlewares') as IMiddlewareList;
appMiddlewares.add(authOnly);
```

If you do not need global middleware list, you can use null class as show in following code,

```
container.add('appMiddlewares', TNullMiddlewareListFactory.create());
```

This will create `TNullMiddlewareList` class instance which basically does nothing.

## Creating middleware with Fano CLI

[Fano CLI](/scaffolding-with-fano-cli) provides middleware creation command `--middleware` to simplify task for creating and setting up middleware with dependency container. Read [Creating Middleware](/scaffolding-with-fano-cli#creating-middleware) section for more information.

## Attaching middleware to route

When you register the controller to route, you can add middleware as shown in following code

```
router.get(
    '/hi/{name}',
    hiController
).add(authOnly);

router.post(
    '/hi/{name}',
    hiController
).add(ajaxOnly)
.add(authOnly);
```

## Built-in middlewares

Fano Framework provides several built-in middlewares.

- `TNullMiddleware`, middleware class that does nothing and just pass the request.
- `TCompositeMiddleware`, middleware class which group several middlewares as one.
- `TRequestHandlerAsMiddleware`, adapter middleware which can turn request handler as a middleware.
- `TCorsMiddleware`, middleware class which adds CORS response header. Read [Handling CORS](/security/handling-cors) for more information.
- `TCsrfMiddleware`, middleware class which adds CSRF protection. Read [Cross-Site Request Forgery (CSRF)](/security/csrf-protection) for more information.
- `TValidationMiddleware`, middleware class which validate request.

### Group several middlewares as one

For example if you have following route registration which each of routes is using same
middlewares

```
router.get(
    '/hello/{name}',
    helloController
).add(cors)
.add(authOnly)
.add(ajaxOnly);

router.get(
    '/hi/{name}',
    hiController
).add(cors)
.add(authOnly)
.add(ajaxOnly);
```

You can simplify it to become

```
router.get(
    '/hello/{name}',
    helloController
).add(corsAuthAjax);

router.get(
    '/hi/{name}',
    hiController
).add(corsAuthAjax);
```

where `corsAuthAjax` middleware is defined as follows

```
var corsAuthAjax : IMiddleware;
...
corsAuthAjax := TCompositeMiddleware.create([cors, authOnly, ajaxOnly]);
```

### Use controller as a middleware

```
var helloCtrlMiddleware : IMiddleware;
    helloContrller : IController;
...
helloCtrlMiddleware := TRequestHandlerAsMiddleware.create(helloController);
```

## Explore more

- [Middleware example application](/examples)
- [Dispatcher](/dispatcher)
- [Working with Router](/working-with-router)
