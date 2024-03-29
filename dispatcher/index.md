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

Fano Framework comes with several dispatcher implementations.

- `TSimpleDispatcher` is light-weight dispatcher that does not offer middleware layer.
- `TXSimpleDispatcher` is similar to `TSimpleDispatcher` with capability to decorate request, response and CGI environment. This is provided, for example, to [allow HTTP override]((/security/http-verb-tunnelling)) `_method` parameter.
- `TDispatcher` is dispatcher that supports middleware.
- `TXDispatcher` is similar to `TDispatcher` with capability like `TXSimpleDispatcher`.
- `TMwExecDispatcher` is similar to `TXDispatcher` except it makes sure global middlewares are always executed although route does not exist or method verb is not allowed.
- `TMaintenanceModeDispatcher` is decorater dispatcher that makes application enters maintenance mode when a special file exists.
- `TVerbTunnellingDispatcher` is decorator dispatcher that allows web application to serve request through [HTTP verb tunnelling](/security/http-verb-tunnelling).

## Creating dispatcher
If you use [Fano CLI](https://github.com/fanoframework/fano-cli) to [scaffold your web application project](/scaffolding-with-fano-cli), you can skip this as Fano CLI creates dispatcher instance for you.

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

### Maintenance mode

To allow application to enter maintenance mode, use `TMaintenanceModeDispatcher`.
```
var actualDispatcher : IDispatcher;
    maintenanceModeDispatcher : IDispatcher;
...

maintenanceModeDispatcher := TMaintenanceModeDispatcher.create(actualDispatcher);
```

To register maintenance mode dispatcher in dependency container, use
`TMaintenanceModeDispatcherFactory` class.

```
container.add(
    'dispatcher',
    TMaintenanceModeDispatcherFactory.create(
        TVerbTunnellingDispatcherFactory.create(
            TSimpleDispatcherFactory.create(
                router,
                TRequestResponseFactory.create()
            )
        )
    )
);
```

By default, this dispatcher checks if file `__maintenance__` exists in current
directory. If it does then it assumes application is in maintenance mode and
raise `EServiceUnavailable` exception.

To make application enters maintenance mode, create empty file with name
`__maintenance__` in current working directory.  For example

```
$ touch __maintenance__
```
To leave maintenance mode, just remove it.
```
$ rm __maintenance__
```

To use different filename, set its file path using `path()` method of `TMaintenanceModeDispatcherFactory()`.

```
container.add(
    'dispatcher',
    TMaintenanceModeDispatcherFactory.create(
        TVerbTunnellingDispatcherFactory.create(
            TSimpleDispatcherFactory.create(
                router,
                TRequestResponseFactory.create()
            )
        )
    ).path('/home/example/maintenance')
);
```
## <a name="set-dispatcher"></a> Set dispatcher

Fano Framework allows application to change dispatcher implementation to use, by
overriding protected `buildDispatcher()` method of `TBasicServiceProvider` class. In this method implementation, you must returns
instance of dispatcher.

```
function TMyAppServiceProvider.buildDispatcher(const container : IDependencyContainer) : IDispatcher;
begin
    result := container['dispatcher'] as IDispatcher;
end;
```

If you use Fano CLI to [scaffold your web application project](/scaffolding-with-fano-cli), you can declared `buildDispatcher()` in `bootstrap.pas` file as shown in following example.

```
TAppServiceProvider = class(TDaemonAppServiceProvider)
protected
    function buildDispatcher(
        const ctnr : IDependencyContainer;
        const routeMatcher : IRouteMatcher;
        const config : IAppConfiguration
    ) : IDispatcher; override;
    ...
end;
...
function TAppServiceProvider.buildDispatcher(
    const ctnr : IDependencyContainer;
    const routeMatcher : IRouteMatcher;
    const config : IAppConfiguration
) : IDispatcher;
begin
    ctnr.add('appMiddlewares', TMiddlewareListFactory.create());

    ctnr.add(
        'my-dispatcher',
        TDispatcherFactory.create(
            ctnr['appMiddlewares'] as IMiddlewareLinkList,
            routeMatcher,
            TRequestResponseFactory.create()
        )
    );
    result := ctnr['my-dispatcher'] as IDispatcher;
end;
```
See [example of adding dispatcher with middleware support](https://github.com/fanoframework/fano-csrf/blob/master/src/bootstrap.pas).

## Explore more

- [Working with Router](/working-with-router)
- [Dependency container](/dependency-container)
