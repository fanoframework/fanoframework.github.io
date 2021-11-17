---
title: Working with Controllers
description: Tutorial on how to work with controllers in Fano Framework
---

<h1 class="major">Working with Controllers</h1>

<a href="/assets/images/request-response-cycle.svg">
<img src="/assets/images/request-response-cycle.svg" alt="Request response cycle diagram" width="60%">
</a>

Request response cycle diagram.

When web application receives [request](/working-with-request), [dispatcher](/dispatcher) uses uri and HTTP method to [match route](/working-with-router) for the request. If route is found, associated controller will be called, otherwise it raises `ERouteHandlerNotFound` exception. Controller than fetch required data from model. Model may retrieve data from storage such as database system. View builds response output to controller which pass response back to web server and eventually back to client browser.

In Fano Framework, controller is any class implements `IRequestHandler` interface. This is where application business logic resides.

## IRequestHandler interface

`IRequestHandler` interface is basis of request handler implementation in Fano Framework. It consists of `handleRequest()` method that implementor class must provide. Dispatcher invokes this method and passes request, response and route argument objects. It must return instance of [response](/working-with-response). You can return response given by dispatcher or return entirely new response instance.

```
IRequestHandler = interface
    ['{483E0FAB-E1E6-4B8C-B193-F8615E039369}']

    (*!-------------------------------------------
     * handle request
     *--------------------------------------------
     * @param request object represent current request
     * @param response object represent current response
     * @param args object represent current route arguments
     * @return new response
     *--------------------------------------------*)
    function handleRequest(
        const request : IRequest;
        const response : IResponse;
        const args : IRouteArgsReader
    ) : IResponse;
end;
```

- `request`, current request object
- `response`, response object
- `args`, current route arguments. Read [Working with Router](/working-with-router) for more information about purpose of this object.

## Built-in IRequestHandler implementation
Fano Framework provides `TAbstractController`, `TController`,
`TMethodRequestHandler` and `TFuncRequestHandler` class. They implement `IRequestHandler` interface.

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

### TMethodRequestHandler

[`TMethodRequestHandler` is concrete class](https://github.com/fanoframework/fano/blob/master/src/Dispatcher/MethodRequestHandlerImpl.pas). It allows [use of any class method as request handler](#method-function-as-request-handler) as long as it matches [`THandlerMethod`](https://github.com/fanoframework/fano/blob/master/src/Dispatcher/Contracts/HandlerTypes.pas).

### TFuncRequestHandler

[`TFuncRequestHandler` is concrete class](https://github.com/fanoframework/fano/blob/master/src/Dispatcher/FuncRequestHandlerImpl.pas). It allows [use of Pascal function as request handler](#method-function-as-request-handler) as long as it matches [`THandlerFunc`](https://github.com/fanoframework/fano/blob/master/src/Dispatcher/Contracts/HandlerTypes.pas).


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

## Custom IRequestHandler implementation
`TAbstractController` and `TController` are provided for convinience. In fact, you are not required to use them at all. You can use any class as long as it implements `IRequestHandler` interface.

For example, if you prefer to handle CRUD task in a single class instead of multiple class.

```
unit UserController;

interface

{$MODE OBJFPC}
{$H+}

uses

    fano;

type

    TUserController = class(TInterfacedObject, IRequestHandler)
    public
        function createUser(
            const request : IRequest;
            const response : IResponse;
            const args : IRouteArgsReader
        ) : IResponse;

        function showUser(
            const request : IRequest;
            const response : IResponse;
            const args : IRouteArgsReader
        ) : IResponse;

        function updateUser(
            const request : IRequest;
            const response : IResponse;
            const args : IRouteArgsReader
        ) : IResponse;

        function deleteUser(
            const request : IRequest;
            const response : IResponse;
            const args : IRouteArgsReader
        ) : IResponse;

        function handleRequest(
            const request : IRequest;
            const response : IResponse;
            const args : IRouteArgsReader
        ) : IResponse; override;
    end;

implementation

    function TUserController.createUser(
        const request : IRequest;
        const response : IResponse;
        const args : IRouteArgsReader
    ) : IResponse;
    begin
        //handle user creation
        result := response;
    end;

    function TUserController.showUser(
        const request : IRequest;
        const response : IResponse;
        const args : IRouteArgsReader
    ) : IResponse;
    begin
        //handle view user
        result := response;
    end;

    function TUserController.updateUser(
        const request : IRequest;
        const response : IResponse;
        const args : IRouteArgsReader
    ) : IResponse;
    begin
        //handle user update
        result := response;
    end;

    function TUserController.deleteUser(
        const request : IRequest;
        const response : IResponse;
        const args : IRouteArgsReader
    ) : IResponse;
    begin
        //handle user deletion
        result := response;
    end;

    function TUserController.handleRequest(
        const request : IRequest;
        const response : IResponse;
        const args : IRouteArgsReader
    ) : IResponse;
    begin
        result := response;
        case request.method of
            'GET' : result := showUser(request, response, args);
            'POST' : result := createUser(request, response, args);
            'PUT' : result := updateUser(request, response, args);
            'DELETE' : result := deleteUser(request, response, args);
        end;
    end;
end.
```
You register route as follows

```
router.any('/users', TUserController.create());
```

## <a name="method-function-as-request-handler">Use class method or function as request handler
Fano Framework provides `TMethodRequestHandler` and `TFuncRequestHandler` adapter class which implements `IRequestHandler` interface to allow use of method or function as request handler. For example, using `TUserController` class above,

```
var
    userCtrl : TUserController;

router.get(
    '/users/show/{id}',
    TMethodRequestHandler.create(@userCtrl.showUser)
);
router.post(
    '/users/create',
    TMethodRequestHandler.create(@userCtrl.createUser)
);
router.put(
    '/users/update',
    TMethodRequestHandler.create(@userCtrl.updateUser)
);
```
You can also use ordinary function,

```
function hello(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
begin
    //handle view user
    result := response;
end;

router.get(
    '/users/show/{id}',
    TFuncRequestHandler.create(@hello)
);
```

[![TFuncRequestHandler video tutorial](/assets/images/func-as-controller.png){:class="image fit"}](https://youtu.be/-EG27KqI1ao "TFuncRequestHandler video tutorial")

TFuncRequestHandler video tutorial explains how to use this class.

## Explore more

- [Working with Request](/working-with-request)
- [Working with Response](/working-with-response)
- [Working with Views](/working-with-views)
- [Working with Models](/working-with-models)
- [Working with Router](/working-with-router)
- [Video tutorial how to use function as request handler](https://youtu.be/-EG27KqI1ao).
