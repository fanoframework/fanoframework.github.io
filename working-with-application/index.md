---
title: Working with Application
description: Tutorial on how to work with Fano Framework application class.
---
<h1 class="major">Working with Application</h1>

## IWebApplication interface

In Fano Framework, application must implements `IWebApplication` interface.
Currently, it does not provide any method except inherit `run()` method from
`IRunnable` interface which is its parent.

[View IWebApplication source code](https://github.com/fanoframework/fano/blob/master/App/Contracts/AppIntf.pas).

## Built-in Implementation

Fano Framework provides built-in implementations of `IWebApplication`,

- `TCgiWebApplication`, CGI web application that implement CGI protocol. It is basically just shell application which run and then terminate after completing its job.
- `TDaemonWebApplication`, is web application that run forever and listen for request until it is terminated manually. This is used mostly for FastCGI, SCGI and uwsgi web application.

## CGI Application

`TCgiWebApplication` is built-in class that implements CGI protocol. Its task is basically to read CGI environment variables that web server send and call appropriate request handler.

Its constructor method expects two parameter, instance of `IAppServiceProvider` and `IRouteBuilder` interface. The first parameter must provide essentials services that is required by application and also register any application dependencies.

For CGI application, Fano Framework provides abstract class `TBasicAppServiceProvider` which you need to extend to register you application dependencies. You must implements its `register()` method.

The last parameter is instance of class responsible to build application routes. Fano Framework provides base abstract class `TRouteBuilder` which you need to extend. You must implements its `buildRoutes()`. Please note that, you do not required to inherit from `TRouteBuilder` class. You can use any class as long as it implements `IRouteBuilder` interface.

```
appInstance := TCgiWebApplication.create(
    TAppServiceProvider.create(),
    TAppRoutes.create()
);
```

Where `TAppServiceProvider` is declared as follow,

```
    TAppServiceProvider = class(TDaemonAppServiceProvider)
    public
        procedure register(const container : IDependencyContainer); override;
    end;
...
    procedure TAppServiceProvider.register(const container : IDependencyContainer);
    begin
        //register all application required service here
    end;
```

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
    end;

```

[View TCgiWebApplication source code](https://github.com/fanoframework/fano/blob/master/App/Implementations/Cgi/AppImpl.pas).

## FastCGI Application

To create web application that support FastCGI protocol, create application service provider which use `TFastCgiAppServiceProvider` as shown in following code,

```
    appInstance := TDaemonWebApplication.create(
        TFastCgiAppServiceProvider.create(
            TServerAppServiceProvider.create(
                TAppServiceProvider.create(),
                TInetSocketSvr.create(host, port)
            )
        ),
        TAppRoutes.create()
    );
```
See [fano-fastcgi](https://github.com/fanoframework/fano-fastcgi) example demo application.

You can replace `TInetSockSvr` with `TUnixSocketSvr` class if you want to use Unix Domain Socket file instead of TCP port. See [fano-fcgi-unix](https://github.com/fanoframework/fano-fcgi-unix) example demo application.

You can replace `TInetSockSvr` with `TBoundSocketSvr` if you want to create FastCGI application which run and managed by web server, for example Apache with mod_fcgid module. See
[fano-fcgid](https://github.com/fanoframework/fano-fcgid) example demo application.

`TInetSockSvr`, `TUnixSocketSvr` and `TBoundSocketSvr` are using [select()](http://man7.org/linux/man-pages/man2/select.2.html) to monitor if socket is ready for I/O operation. To use [epoll](http://man7.org/linux/man-pages/man7/epoll.7.html), replace with `TEpollInetSocketSvr`, `TEpollUnixSocketSvr` and `TEpollBoundSocketSvr`.

See [Deploy as FastCGI application](/deployment/fastcgi) for information how to
deploy FastCGI application on various web servers.

You may want to read [Scaffolding FastCGI project directory structure](/scaffolding-with-fano-cli/creating-project#scaffolding-fastcgi-project) for information how to scaffolding FastCGI web application with [Fano CLI](https://github.com/fanoframework/fano-cli).

## SCGI Application

To create web application that support [SCGI (Simple Common Gateway Interface)](https://python.ca/scgi/protocol.txt) protocol, create application service provider which use `TScgiAppServiceProvider` as shown in following code,

```
    appInstance := TDaemonWebApplication.create(
        TScgiAppServiceProvider.create(
            TServerAppServiceProvider.create(
                TAppServiceProvider.create(),
                TInetSocketSvr.create(host, port)
            )
        ),
        TAppRoutes.create()
    );
```
See [fano-scgi](https://github.com/fanoframework/fano-scgi) example demo application.

See [Deploy as SCGI application](/deployment/scgi) for information how to
deploy SCGI application on various web servers.

You may want to read [Scaffolding SCGI project directory structure](/scaffolding-with-fano-cli/creating-project#scaffolding-scgi-project) for information how to scaffolding SCGI web application easily.

## Uwsgi Application

To create web application that support [uwsgi](https://uwsgi-docs.readthedocs.io/en/latest/Protocol.html) protocol, create application service provider which use `TUwsgiAppServiceProvider` as shown in following code,

```
    appInstance := TDaemonWebApplication.create(
        TUwsgiAppServiceProvider.create(
            TServerAppServiceProvider.create(
                TAppServiceProvider.create(),
                TInetSocketSvr.create(host, port)
            )
        ),
        TAppRoutes.create()
    );
```
See [fano-uwsgi](https://github.com/fanoframework/fano-uwsgi) example demo application.

See [Deploy as uwsgi application](/deployment/uwsgi) for information how to
deploy uwsgi application on various web servers.

You may want to read
[Scaffolding uwsgi project directory structure](/scaffolding-with-fano-cli/creating-project#scaffolding-uwsgi-project) for information how to scaffolding uwsgi web application easily.

## Apache modules Application

Implementation of IWebApplication that run as Apache modules is not yet implemented.

## Standalone web server application

Implementation of IWebApplication that run as standalone web server is not yet implemented.

## Application implementation which support Rack-like protocol

Implementation of IWebApplication that implements Rack-like protocol, currently, is still under development.

- [Ruby Rack](https://rack.github.io/)
- [Prack](https://github.com/piradoiv/Prack)

## How to choose IWebApplication implementation?

### CGI Application

#### Pros

- Easy to setup especially on shared-hosting where you do not posses server administrive privilege.

### Cons

- Not very performant compared to FastCGI or SCGI application due to how CGI protocol works.

### FastCGI Application

#### Pros

- Performance is good compared to CGI application.

### Cons

- Requires server administrative privilege to setup. You can not deploy application on shared-hosting easily.

### SCGI Application

#### Pros

- Performance is good compared to CGI application and maybe slightly better than
FastCGI due to simpler protocol specification.

### Cons

- Requires server administrative privilege to setup. You can not deploy application on shared-hosting easily.

### uwsgi Application

#### Pros

- Performance is comparable to SCGI protocol but less network bandwidth requirement due to use of binary protocol.

### Cons

- Requires server administrative privilege to setup. You can not deploy application on shared-hosting easily.

## Explore more

- [Deployment](/deployment)
- [Deploy CGI Application](/deployment/cgi)
- [Deploy FastCGI Application](/deployment/fastcgi)
- [Deploy SCGI Application](/deployment/scgi)
- [Deploy uwsgi Application](/deployment/uwsgi)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
