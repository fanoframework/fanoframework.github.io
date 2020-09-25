---
title: Working with Router
description: Tutorial on how to work with router in Fano Framework
---

<h1 class="major">Working with Router</h1>

## Router and route

In Fano Framework, a route is an association rule between URL path pattern, HTTP method and code that handles it. Router manages one or more routes and match request URL path, extract data in it and select code that handles it. Router is any class implements `IRouter` interface.

### Creating route for GET method

```
var
    router : IRouter;
    handler : IRequestHandler;
...
router.get('/', handler);
```

### Creating route for POST method

```
router.post('/submit', handler);
```

### Creating route for PUT method

```
router.put('/submit', handler);
```

### Creating route for DELETE method

```
router.delete('/submit', handler);
```

### Creating route for PATCH method

```
router.patch('/submit', handler);
```

### Creating route for HEAD method

```
router.head('/', handler);
```

### Creating route for OPTIONS method

```
router.options('/', handler);
```

### Creating route for multiple methods

```
router.map(['GET', 'POST'], '/', handler);
```

### Creating route for all methods

```
router.any('/', handler);
```
### Route with argument
A route can have argument as shown in following example.
```
router.get('/user/{username}', myUserHandler);
```
Read [Getting Route Argument](#getting-route-argument) section for information how to read route argument value.

## Route matching
If we have following route setup

```
var
    router : IRouter;
    myAppHandler, anotherAppHandler : IRequestHandler;
...

//GET request
router.get('/my/app', myAppHandler);

//POST request
router.post('/another/app', anotherAppHandler);
```

If your application hostname is `example.com` and  client opens `http://example.com/my/app` through browser, our application will receive request `GET` method to `/my/app` resources. Router will match HTTP method and URL and returns `myAppHandler` as code that responsible to handle this request.

If client opens `http://example.com/another/app` through browser, our application will receive request `GET` method to `/another/app` resources. Router will find match to `/another/app` but because it is only registered for `POST` request, `EMethodNotAllowed` exception will be raised with HTTP 405 error code.

If client opens `http://example.com/not/exists` through browser, our application will receive request `GET` method to `/not/exists` resources. Router will not find any matches. If this happens, `ERouteHandlerNotFound` exception will be raised with HTTP 404 error code.

## <a name="route-builder"></a>Build application routes with route builder

To build application routes, you need to create class that implements `IRouteBuilder` interface. 
```
IRouteBuilder = interface
    ['{46016416-B249-4258-B76A-7F5B55E8348D}']

    (*!----------------------------------------------
        * build application routes
        * ----------------------------------------------
        * @param cntr instance of dependency container
        * @param rtr instance of router
        *-----------------------------------------------*)
    procedure buildRoutes(const cntr : IDependencyContainer; const rtr : IRouter);
end;
```
Fano Framework provides base abstract class `TRouteBuilder` which you can extend and implement its `buildRoutes()` method and pass it when creating application instance as shown in following code,

```
TAppRoutes = class(TRouteBuilder)
public
    procedure buildRoutes(
        const container : IDependencyContainer;
        const router : IRouter
    ); override;
end;
...
procedure TAppRoutes.buildRoutes(
    const container : IDependencyContainer;
    const router : IRouter
);
begin
   //register all routes here
   router.get('/', handler);
end;

```
And then create its instance and pass it to application constructor.
```
appInstance := TCgiWebApplication.create(
    TAppServiceProvider.create(),
    TAppRoutes.create()
);
```

If you use [Fano CLI to scaffold web application](/scaffolding-with-fano-cli), you may notice that routes are separated into one or more include files that are inserted in one class responsible to build application routes as shown in following example,

```
procedure TAppRoutes.buildRoutes(
    const container : IDependencyContainer;
    const router : IRouter
);
begin
    {$INCLUDE Routes/routes.inc}
end;
```

This is just convention used by Fano CLI tools because it is simple to generate.

Fano Framework cares that you provide class that implements `IRouteBuilder`. It does not care how you implement it. So you are free to compose route builder class the way it suits you.

For example, you can create separate `IRouteBuilder` implementation for each feature for better code organization and then compose them using `TCompositeRouteBuilder` class as shown in example below

```
TUsersRoutes = class(TRouteBuilder)
public
    procedure buildRoutes(
        const container : IDependencyContainer;
        const router : IRouter
    ); override;
end;
...
procedure TUsersRoutes.buildRoutes(
    const container : IDependencyContainer;
    const router : IRouter
);
begin
    //register users related routes here
end;
```
For product feature routes,

```
TProductRoutes = class(TRouteBuilder)
public
    procedure buildRoutes(
        const container : IDependencyContainer;
        const router : IRouter
    ); override;
end;
...
procedure TProductRoutes.buildRoutes(
    const container : IDependencyContainer;
    const router : IRouter
);
begin
    //register products related routes here
end;
```

And when you build application, you compose them as follows,

```
appInstance := TCgiWebApplication.create(
    TAppServiceProvider.create(),
    TCompositeRouteBuilder.create([
        TUserRoutes.create(),
        TProductRoutes.create()
    ]);
);
```

Please note that `TCompositeRouteBuilder` is built-in implementation of `IRouteBuilder` which composes one or more `IRouteBuilder` instances as one.

## Set route name or middlewares with IRoute interface

All methods which register request handler such as `get()`, `post()`, etc., returns
instance of `IRoute` interface which you can use to assign name to route or attach a middlewares. Following code lists all methods in this interface.

```
function setName(const routeName : shortstring) : IRoute;
function getName() : shortstring;
function add(const amiddleware : IMiddleware) : IRoute;
```

For example to set route name and attach a middleware.

```
var authOnlyMiddleware : IMiddleware;
...
router.post('/user/submit', handler).setName('create-user').add(authOnlyMiddleware);
```

Read [Middlewares](/middlewares) for more information.

## <a name="getting-route-argument"></a>Getting route argument

Third parameter of `handleRequest()` method of `IRequestHandler` interface gives instance of `IRouteArgsReader` interface to allow application to retrieve route argument.

If you have route pattern `/myroute/{name}` and you access route via URl `http://example.com/myroute/john` then you can get value of `name` parameter as follows

```
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
var
    arg : TPlaceholder;
begin
    arg := args.getArg('name');
    //arg.name = 'name', arg.value = 'john'
end;
```

If you are interested only for its value, you can call `getValue()` and pass name of argument. It will return value as string.

```
writeln(args.getValue('name'));
```
or with simplified array-like syntax,

```
writeln(args['name']);
```

Above codes will print identical output as follows

```
writeln(args.getArg('name').value);
```

You can get all route arguments using `getArgs()` method,

```
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
var placeHolders : TArrayOfPlaceholders;
    arg : TPlaceholder;
    i:integer;
begin
    placeHolders := args.getArgs();

    for i:=0 to length(placeholders)-1 do
    begin
        arg := placeholders[i];
    end;
end;
```

See [code example](https://github.com/fanoframework/fano-app/blob/master/app/App/Hello/Controllers/HelloController.pas) how to read route argument.

## Create router instance

Fano Framework comes with basic router implementation `TRouter` class which implements `IRouter` interface.

```
container.add('router', TSimpleRouterFactory.create());
```
`TSimpleRouterFactory` class builds router instance that supports route argument parsing. Alternatively, you can use `TRouterFactory` class which creates router instance that does not support route argument but it is faster when matching request URL.

```
container.add('router', TRouterFactory.create());
```
If you need only static URL path pattern, you should use it. 

If you create application service provider inherit from `TBasicAppServiceProvider`, it will create default router using `TSimpleRouterFactory` class which is good enough for most application. 

## Replace router instance
If you want to replace router with different implementation, you can override `buildRouter()` method of `TBasicAppServiceProvider`. For example,

```
TMyAppProvider = class(TBasicAppServiceProvider)
private
    fRouterMatcher : IRouteMatcher;
public
    function getRouteMatcher() : IRouteMatcher; override;
    function buildRouter(const cntr : IDependencyContainer) : IRouter; override;
end;
...
function TMyAppProvider.buildRouter(const cntr : IDependencyContainer) : IRouter;
begin
    ctnr.add('router', TRouterFactory.create());
    result := ctnr['router'] as IRouter;
    fRouteMatcher := result as IRouteMatcher;
end;    

function TMyAppProvider.getRouteMatcher() : IRouteMatcher;
begin
    result := fRouteMatcher;
end;    
```
Note that `IRouteMatcher` is interface which is responsible to match request URL and `TRouter` implements it.

## Explore more

- [Dispatcher](/dispatcher)
- [Middlewares](/middlewares)
