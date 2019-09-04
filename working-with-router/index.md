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
    myAppHandler, anotherAppHandler : IRouteHandler;
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
    homeRouteHandler : IRouteHandler;
...
router.get('/', homeRouteHandler);
```

### Creating route for POST method

```
var handler : IRouteHandler;
...
router.post('/submit', handler);
```

### Creating route for PUT method

```
var handler : IRouteHandler;
...
router.put('/submit', handler);
```

### Creating route for DELETE method

```
var submitHandler : IRouteHandler;
...
router.delete('/submit', submitHandler);
```

### Creating route for PATCH method

```
var submitHandler : IRouteHandler;
...
router.patch('/submit', submitHandler);
```

### Creating route for HEAD method

```
var handler : IRouteHandler;
...
router.head('/', handler);
```

### Creating route for OPTIONS method

```
var handler : IRouteHandler;
...
router.options('/', handler);
```

### Creating route for multiple methods

```
var handler : IRouteHandler;
...
router.map(['GET', 'POST'], '/', handler);
```

### Creating route for all methods

```
var handler : IRouteHandler;
...
router.any('/', handler);
```

## Getting route argument

`IRouteHandler` interface provides `getArg()` to allow application to retrieve route argument.

If you have route pattern `/myroute/{name}` and you access route via URl `http://[your-host-name]/myroute/john` then you can get value of `name` parameter as follows

```
var handler : IRouteHandler;
    arg : TPlaceholder;
...
arg := handler.getArg('name');
//arg.phName = 'name', arg.phValue = 'john'
```

You can get all route arguments using `getArgs()` method,

```
var placeHolders : TArrayOfPlaceholders;
    arg : TPlaceholder;
    i:integer;
...
placeHolders := getArgs();

for i:=0 to length(placeholders)-1 do
begin
    arg := placeholders[i];
end;
```

See [code example](https://github.com/fanoframework/fano-app/blob/master/app/App/Hello/Controllers/HelloController.pas) how to read route argument.

## Explore more

- [Dispatcher](/dispatcher)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
