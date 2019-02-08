---
title: Working with Controllers
description: Tutorial on how to work with controllers in Fano Framework
---

<h1 class="major">Working with Controllers</h1>

## IRequestHandler and IRouteHandler interface

When web application receives request from web server, it will be given
request uri of resource that user is requesting.
Dispatcher parses request uri to find matching route handler that will handle this request. If no handler available to handle it, HTTP 404 will be returned.

Interface `IRequestHandler`, declared in unit `fano.pas`, is basis of route handler implementation in Fano Framework. It consists of `handleRequest()` method that implementor class must provide.

Interface `IRouteHandler`, is extension to `IRequestHandler` interface and provide more functionality such get route argument data or set middleware for route.

`IRouteHandler` is basis of all route handler. When you set route, you must pass instance of class that implements `IRouteHandler`.

Fano Framework provides `TRouteHandler` as base abstract class that implements `IRouteHandler` and also base controller in `TController` class which derived from `TRouteHandler` class.

`TRouteHandler` is abstract class. You need to derive and implements its `handleRequest()` method to be able to use it. This class is useful, for example, in following situation:

- you do not need to use view
- you prefer to compose response by yourself, for example, to output JPEG image response.

`TController` is concrete class and extends `TRouteHandler` capability by adding view and view parameters to allow, for example, to use template.

But of course, you are free to implements your own.

## Using TController class

`TController` class is built-in class that provides ability for route handler to works with view and middlewares.

Except for simple route handler which display static view, you are very likely need to extends this class as `TController` by defaut, does few things, which is
returning response from `IView` instance.

## Creating TController class

`TController` constructor expects 4 parameters

```
constructor TController.create(
    const beforeMiddlewares : IMiddlewareCollection;
    const afterMiddlewares : IMiddlewareCollection;
    const viewInst : IView;
    const viewParamsInst : IViewParameters
);
```

- `beforeMiddlewares`, instance of collection of middlewares that will be called
before this controller get executed.
- `afterMiddlewares`, instance of collection of middlewares that will be called
after this controller get executed.
- `viewInst`, view to be used, i.e., instance of class that implements `IView` interface.
- `ViewParamsInst`, view parameters, i.e., instance of class that implements `IViewParameters` interface.

`viewInst` and `viewParamsInst` that you pass during class construction, will be available from inherited class as `gView` and `viewParams` respectively.

For more information regarding view and view parameters, read [Working with Views](/working-with-views).

## Implements controller logic

`handleRequest()` is method that will be invoked by dispatcher to handle request, so you mostly do not call it directly.
This method is part of `IRequestHandler` interface. Dispatcher will pass request and response instance to this method.

```
function handleRequest(
    const request : IRequest;
    const response : IResponse
) : IResponse;
```

- `request` is current request instance
- `response`, current response instance

`handleRequest()` should return instance of response that will be used as response to request. You can return response given by dispatcher or create entirely new response instance.


`TController` class provides basic implementation of this method, which is, to return view output. Fano Framework provides some built-in response class that you can use such HTML response, JSON response or binary response (for example to output image). Of course, you are free to implements your own output response.
