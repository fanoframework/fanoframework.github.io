---
title: Working with Router
description: Tutorial on how to work with router in Fano Framework
---

<h1 class="major">Working with Router</h1>

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

## Getting route argument

`IRouteHandler` interface provides `getArg()` to allow application to retrieve route argument.

If you have route pattern `/myroute/{name}` and you access route via URl `http://[your-host-name]/myroute/john` then you can get value of `name` parameter as follows

```
var handler : IRouteHandler;
    name : string;
...
name := handler.getArg('name'); //name = 'john'
```
