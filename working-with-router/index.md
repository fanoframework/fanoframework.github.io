---
title: Working with Router
description: Tutorial on how to work with router in Fano Framework
---

<h1 class="major">Working with Router</h1>

## What is router?

In Fano Framework, router is instance responsible to map a request to a code that handles it based on certain rule.

It is similar to front desk staff in a building. When a customer (request) comes in to building looking for an accounting staff (URL), first, she/he checks standard operational procedure (rule) to check whether is ok for outsider to contact accounting staff. If there are rule allows it, she/he directs the customer to accounting departement.

Similarly, when HTTP request is made to server, router determines code to handle request based on URL and HTTP method.

## About route

In Fano Framework, a route is an association rule between a request (identified by HTTP method and URL) and code that handles it.

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

If client opens `http://[app hostname]/my/app` through browser, our application will receive request `GET` method to `/my/app` resources. Router will match HTTP method and URL and returns `myAppHandler` as code that responsible to handle this request.

If client opens `http://[app hostname]/another/app` through browser, our application will receive request `GET` method to `/another/app` resources. Router will find match to `/another/app` but because it is only registered for `POST` request, exception `EMethodNotAllowed` will be raised.

If client opens `http://[app hostname]/not/exists` through browser, our application will receive request `GET` method to `/not/exists` resources. Router will not find any matches. If this happens, exception `EMethodNotAllowed` will be raised.

## Create router instance

Fano Framework comes with basic router implementation `TRouter` class which implements `IRouter` interface.

```
container.add('router', TSimpleRouterFactory.create());
```

## Create route

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

## Set route name or middlewares with IRoute interface

All methods which register request handler such as `get()`, `post()`,.. etc returns
instance of `IRoute` interface which you can use to assign name to route or attach a middlewares. Following code lists all methods in this interface.

```
function setName(const routeName : shortstring) : IRoute;
function getName() : shortstring;
function before(const amiddleware : IMiddleware) : IRoute;
function after(const amiddleware : IMiddleware) : IRoute;
```

For example to set route name and attach a middleware.

```
var authOnlyMiddleware : IMiddleware;
...
router.post('/user/submit', handler).setName('create-user').before(authOnlyMiddleware);
```

Read [Middlewares](/middlewares) for more information.

## Getting route argument

Third parameter of `handleRequest()` method of `IRequestHandler` interface gives instance of `IRouteArgsReader` interface to allow application to retrieve route argument.

If you have route pattern `/myroute/{name}` and you access route via URl `http://[your-host-name]/myroute/john` then you can get value of `name` parameter as follows

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

## Explore more

- [Dispatcher](/dispatcher)
- [Middlewares](/middlewares)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
