---
title: Dispatcher
description: Request dispatcher in Fano Framework
---

<h1 class="major">Dispatcher</h1>

## Dispatcher task

In general, task of a dispatcher is quite simple. Given request URI and HTTP method, it asks router to find instance of class that will handle the request, build middlewares, passing all relevant information and execute it.

A dispatcher in Fano Framework must implements `IDispatcher` interface.

```
IDispatcher = interface
    ['{F13A78C0-3A00-4E19-8C84-B6A7A77A3B25}']

    (*!-------------------------------------------
        * dispatch request
        *--------------------------------------------
        * @param env CGI environment
        * @param stdIn STDIN reader
        * @return response
        *--------------------------------------------*)
    function dispatchRequest(
        const env: ICGIEnvironment;
        const stdIn : IStdIn
    ) : IResponse;
end;
```
- `env`, CGI environment variable that is given by web server.
- `stdIn`, object which capable of reading STDIN

## Built-in Dispatcher implementation

Fano Framework comes with two dispatcher implementation, `TSimpleDispatcher` and
`TDispatcher` class.

`TSimpleDispatcher` is light-weight dispatcher that does not offer middleware layer
while the latter supports middleware.

## Creating dispatcher

Creating simple dispatcher:

```
var router : IRouteMatcher;
...
container.add('dispatcher', TSimpleDispatcherFactory.create(router));
```

Creating basic dispatcher with middleware support

```
var router : IRouteMatcher;
...
container.add('dispatcher', TDispatcherFactory.create(router));
```

## Set dispatcher

Fano Framework allows application to change dispatcher implementation to use and
can be set in `initDispatcher()`. In this method implementation, you must returns
instance of dispatcher.

```
function TMyApp.initDispatcher(const container : IDependencyContainer) : IDispatcher;
begin
    result := container.get('dispatcher') as IDispatcher;
end;
```

## Explore more

- [Working with Router](/working-with-router)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
