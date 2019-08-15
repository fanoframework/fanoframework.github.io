---
title: Middlewares
description: Documentation about how middlewares work in Fano Framework
---

<h1 class="major">Middlewares</h1>

## Why middleware?

In Fano Framework, middleware is an optional software component that is executed before or after request is passed to actual request handler. It is similar concept to firewall, in which, it can pass, block or modify request or modify response.

For example, middleware allows developer to test if user is logged in before request reaches controller. If user is not logged in, it blocks request.
So when controller is executed, developer can be sure that user must be logged in.

Single middleware instance can be attach to one or more route. This allows centralized action to be taken for multiple controllers.

## Middleware architecture in Fano Framework

Fano Framework use simple chained middleware list. Each middleware can decide whether to pass request to next middleware or block. If middleware blocks a request, it must return response

[client] <--> [bm-0] <--> [bm-1] <--> ... <--> [bm-n] <--> [controller] <--> [am-0] <--> [am-1] <--> ... <--> [am-n]

## Type of middlewares

### Based on scope

#### Global middleware

Global middleware is middleware that is attached and applied globally to all routes.

#### Route middleware

Route middleware is middleware that is attached and applied to one or more specific routes.

### Based on order of execution

### Before middleware

Before middleware is any middleware that is executed *before* controller execution. This is mostly type of middleware that can be used modify request or act as gate that block or pass request to

### After middleware

After middleware is any middleware that is executed *after* controller execution.


## Creating middleware

```
var ajaxOnly : IMiddleware;
    authOnly : IMiddleware;
...
ajaxOnly := TAjaxOnlyMiddleware.create();
authOnly := TAuthOnlyMiddleware.create();
```

## Attaching middleware to global middleware

When you use initialize `IDispatcher` implementation which support middlewares, such as `TDispatcher` class, you are required to setup two `IMiddlewareCollectionAware` instances. One is for global middlewares (single instance) and the other for per-route middleware (multiple instance) (See *Single vs Multiple instance* section in [Dependency Container](/depdendency-container)),

As shown in following code

```
{-----------------------------------------------
  register middleware list for application
------------------------------------------------}
container.add('appMiddlewares', TMiddlewareCollectionAwareFactory.create());

{-----------------------------------------------
  register middleware list for each routes
  need to be use factory so each route will have
  different middleware list
------------------------------------------------}
container.factory('routeMiddlewares', TMiddlewareCollectionAwareFactory.create());
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
        container.get('appMiddlewares') as IMiddlewareCollectionAware,
        aRouterInst as IRouteMatcher
    )
);
```

and then you can register a middleware to global middlewares as follows

```
var appMiddlewares : IMiddlewareCollectionAware;
...
appMiddlewares := container.get('appMiddlewares') as IMiddlewareCollectionAware;
appMiddlewares.getMiddlewares().addBefore(authOnly);
```

## Attaching middleware to route

When controller or route hander is created, you need to pass per-route middleware collection

```
function THiControllerFactory.build(const container : IDependencyContainer) : IDependency;
var routeMiddlewares : IMiddlewareCollectionAware;
begin
    routeMiddlewares := container.get('routeMiddlewares') as IMiddlewareCollectionAware;
    try
        result := THiController.create(routeMiddlewares);
    finally
        routeMiddlewares := nil;
    end;
end;

```

and when you register the controller to route, you can add middleware as shown in following code

```
(router.get(
    '/hi/{name}',
    hiController
) as IMiddlewareCollectionAware).getMiddlewares().addBefore(authOnly);

(router.post(
    '/hi/{name}',
    hiController
) as IMiddlewareCollectionAware).getMiddlewares()
    .addBefore(ajaxOnly)
    .addBefore(authOnly);
```

## Explore more

- [Middleware example application](/examples)
- [Dispatcher](/dispatcher)
- [Working with Router](/working-with-router)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
