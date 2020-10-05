---
title: Dispatcher
description: Request dispatcher in Fano Framework
---

<h1 class="major">Dispatcher</h1>

## Dispatcher task

In general, task of a dispatcher is quite simple. Given request URI and HTTP method, it asks [router](/working-with-router) to find instance of class that will handle the request, build [middlewares](/middlewares), passing all relevant information and execute it.

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
- `env`, CGI environment variable that is given by web server. Read [CGI Environment](/environment) for more information.
- `stdIn`, object which capable of reading STDIN

Fano Framework will pass instance of CGI environment and `IStdIn` instance based on protocol that your application use. For example, using CGI or FastCGI protocol, will result different `IStdin` implementation. However, this should be transparent to developer.

## Built-in Dispatcher implementation

Fano Framework comes with two dispatcher implementation, `TSimpleDispatcher` and
`TDispatcher` class.

`TSimpleDispatcher` is light-weight dispatcher that does not offer middleware layer
while the latter supports middleware.

`TVerbTunnellingDispatcher` is decorator dispatcher that allows web application to serve request through [HTTP verb tunnelling](/security/http-verb-tunnelling).

## Creating dispatcher
If you use [Fano CLI](https://github.com/fanoframework/fano-cli) to [scaffold your web application project](/scaffolding-with-fano-cli), you can skip this as Fano CLI create dispatcher instance for you.

Creating simple dispatcher:

```
var router : IRouteMatcher;
...
container.add(
    'dispatcher',
    TSimpleDispatcherFactory.create(
        router,
        TRequestResponseFactory.create()
    )
);
```

For creating basic dispatcher with middleware support, you need to pass instance of `IMiddlewareLinkList` instance. Please read [Middlewares](/middlewares) for more information. `container` is an instance of `IDependencyContainer`. Read [Dependency Container](/dependency-container) for more informations.

```
var router : IRouteMatcher;
...
container.add(
    'dispatcher',
    TDispatcherFactory.create(
        container.get('appMiddlewares') as IMiddlewareLinkList,
        router,
        TRequestResponseFactory.create()
    )
);
```

For creating dispatcher with session support, you need to use `TSessionDispatcherFactory` as shown in following code. Because session support in Fano Framework is implemented using  middleware infrastructure, you also need to pass instance of `IMiddlewareLinkList` instance. Please read [Working with Session](/working-with-session) for more information.

```
var router : IRouteMatcher;
...
container.add(
    'dispatcher',
    TSessionDispatcherFactory.create(
        container.get('appMiddlewares') as IMiddlewareLinkList,
        router,
        TRequestResponseFactory.create(),
        container.get('sessionManager') as ISessionManager,
        (TCookieFactory.create()).domain(config.getString('cookie.domain')),
        config.getInt('cookie.maxAge')
    )
);
```

To allow [HTTP verb tunnelling](/security/http-verb-tunnelling), wrap actual dispatcher factory with `TVerbTunnellingDispatcherFactory` as shown in following code,

```
container.add(
    'dispatcher',
    TVerbTunnellingDispatcherFactory.create(
        TSimpleDispatcherFactory.create(
            router,
            TRequestResponseFactory.create()
        )
    )
);
```
## Set dispatcher

Fano Framework allows application to change dispatcher implementation to use by
overriding `buildDispatcher()` method of `TBasicServiceProvider` class. In this method implementation, you must returns
instance of dispatcher.

```
function TMyAppServiceProvider.buildDispatcher(const container : IDependencyContainer) : IDispatcher;
begin
    result := container['dispatcher'] as IDispatcher;
end;
```

## Explore more

- [Working with Router](/working-with-router)
- [Dependency container](/dependency-container)
