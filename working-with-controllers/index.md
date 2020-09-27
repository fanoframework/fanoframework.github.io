---
title: Working with Controllers
description: Tutorial on how to work with controllers in Fano Framework
---

<h1 class="major">Working with Controllers</h1>

## IRequestHandler interface

When web application receives [request](/working-with-request), dispatcher uses uri and HTTP method to [match route](/working-with-router) for the request. If route is found, associated request handler will be called, otherwise it raises `ERouteHandlerNotFound` exception.

`IRequestHandler` interface is basis of request handler implementation in Fano Framework. It consists of `handleRequest()` method that implementor class must provide. Dispatcher invokes this method and passes request, response and route argument objects. It must return instance of [response](/working-with-response). You can return response given by dispatcher or return entirely new response instance.

```
function handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
```

- `request`, current request object
- `response`, response object
- `args`, current route arguments. Read [Working with Router](/working-with-router) for more information about purpose of this object.

## Built-in IRequestHandler implementation
Fano Framework provides `TAbstractController` and `TController` class.

### TAbstractController
[`TAbstractController` is an abstract class](https://github.com/fanoframework/fano/blob/master/src/Mvc/Controllers/AbstractControllerImpl.pas). You need to derive and implements its `handleRequest()` method to be able to use it.

```
unit MyController;

interface

{$MODE OBJFPC}
{$H+}

uses

    fano;

type

    TMyController = class(TAbstractController)
    public
        function handleRequest(
            const request : IRequest;
            const response : IResponse;
            const args : IRouteArgsReader
        ) : IResponse; override;
    end;

implementation

    function TMyController.handleRequest(
        const request : IRequest;
        const response : IResponse;
        const args : IRouteArgsReader
    ) : IResponse;
    begin
        response.body().write('<html>');
        response.body().write('<head><title>My Controller</title></head>');
        response.body().write('<body><h1>My Controller</h1></body>');
        response.body().write('</html>');
        result := response;
    end;
end.
```
### TController
[`TController` is concrete class](https://github.com/fanoframework/fano/blob/master/src/Mvc/Controllers/ControllerImpl.pas). It extends `TAbstractController` capability by adding view and view parameters to allow, for example, to use template.

But of course, you are free to implements your own. In fact, you are not required to use `TAbstractController` or `TController` at all. You can use any class as long as it implements `IRequestHandler` interface.

## Using TController class

`TController` class is built-in class that provides ability for route handler to works with view and view parameters.

Except for simple route handler which display static view, very likely, you need to extend this class. `TController` by default, just calls `render()` method of `IView`, which is
returning response from `IView` instance.

## Creating TController class

`TController` constructor expects 2 parameters

```
constructor TController.create(
    const viewInst : IView;
    const viewParamsInst : IViewParameters
);
```
- `viewInst`, view to be used, i.e., instance of class that implements `IView` interface.
- `ViewParamsInst`, view parameters, i.e., instance of class that implements `IViewParameters` interface.

`viewInst` and `viewParamsInst` that you pass during class construction, will be available from inherited class as `fView` and `fViewParams` protected fields, respectively.

For more information regarding view and view parameters, read [Working with Views](/working-with-views).

## Implements controller logic

`handleRequest()` is method that will be invoked by dispatcher to handle request, so you mostly do not call it directly.
This method is part of `IRequestHandler` interface. Dispatcher will pass request and response instance to this method.

`TController` class provides basic implementation of this method, which is, to return view output. Fano Framework provides some built-in response class that you can use such HTML response, JSON response or binary response (for example to output image). Of course, you are free to implements your own output response.

## Explore more

- [Working with Request](/working-with-request)
- [Working with Response](/working-with-response)
- [Working with Views](/working-with-views)
- [Working with Models](/working-with-models)
- [Working with Router](/working-with-router)
