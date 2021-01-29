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

## IAppServiceProvider interface

`TCgiWebApplication`  depends on instance of `IAppServiceProvider` interface while `TDaemonWebApplication` depends on `IDaemonAppServiceProvider`. They provide essential services for application. Following code is declaration of this interface.

```
type

    IAppServiceProvider = interface(IServiceProvider)
        ['{41032A70-F31D-45A6-AE26-574888BE4D07}']
        function getContainer() : IDependencyContainer;
        property container : IDependencyContainer read getContainer;

        function getErrorHandler() : IErrorHandler;
        property errorHandler : IErrorHandler read getErrorHandler;

        function getDispatcher() : IDispatcher;
        property dispatcher : IDispatcher read getDispatcher;

        function getEnvironment() : ICGIEnvironment;
        property env : ICGIEnvironment read getEnvironment;

        function getRouter() : IRouter;
        property router : IRouter read getRouter;

        function getStdIn() : IStdIn;
        property stdIn : IStdIn read getStdIn;
    end;

    IDaemonAppServiceProvider = interface(IAppServiceProvider)
        ['{4FF6129E-171F-429C-BE5B-7A8B941D3626}']

        function getServer() : IRunnableWithDataNotif;
        property server : IRunnableWithDataNotif read getServer;

        function getProtocol() : IProtocolProcessor;
        property protocol : IProtocolProcessor read getProtocol;

        function getOutputBuffer() : IOutputBuffer;
        property outputBuffer : IOutputBuffer read getOutputBuffer;

        function getStdOut() : IStdOut;
        property stdOut : IStdOut read getStdOut;

    end;
```

Fano Framework provides several built-in implementation of this interface that you can use or extend.

- `TBasicAppServiceProvider`, abstract class that implements `IAppServiceProvider` interface. Most of methods already has implementation, except its `register()` method which you must implement to register your application specific dependencies. For CGI web application, this class is what you need to use.

- `TDaemonAppServiceProvider`, abstract class inherits from `TBasicAppServiceProvider` class that implements `IDaemonAppServiceProvider` interface. Most of methods already has implementation, except its `register()` method which you must implement to register your application specific dependencies. For daemon web application, this class is what you need to use.

- `TServerAppServiceProvider`, class which provides worker server for daemon web application.

- `TFastCgiAppServiceProvider`, class which provides essentials service for FastCGI web application.

- `TScgiAppServiceProvider`, class which provides essentials service for SCGI web application.

- `TUwsgiAppServiceProvider`, class which provides essentials service for uwsgi web application.

- `TMhdAppServiceProvider`, class which provides essentials service for libmicrohttpd web application.

## IRouteBuilder interface

When creating application instance, you need also to implement `IRouteBuilder` interface which responsible to setup application routes. Please read [Route Builder](/working-with-router#route-builder) section documentation for more information.

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
    TAppServiceProvider = class(TBasicAppServiceProvider)
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

## Daemon web application

Web application that runs forever in background must setup service provider that inherit from `TDaemonAppServiceProvider` such as web application that utilize FastCGI, SCGI or uwsgi  protocol because application depends on `IDaemonAppServiceProvider` interface.

```
    TAppServiceProvider = class(TDaemonAppServiceProvider)
    public
        procedure register(const container : IDependencyContainer); override;
    end;
```

## FastCGI Application

To create web application that support FastCGI protocol, create application service provider which use `TFastCgiAppServiceProvider` as shown in following code,

```
    appInstance := TDaemonWebApplication.create(
        TFastCgiAppServiceProvider.create(
            TServerAppServiceProvider.create(
                TAppServiceProvider.create(),
                (TInetSvrFactory.create(host, port) as ISocketSvrFactory).build()
            )
        ),
        TAppRoutes.create()
    );
```
See [fano-fastcgi](https://github.com/fanoframework/fano-fastcgi) example demo application.

You can replace `TInetSvrFactory` with `TUnixSvrFactory` class if you want to use Unix Domain Socket file instead of TCP port. See [fano-fcgi-unix](https://github.com/fanoframework/fano-fcgi-unix) example demo application.

You can replace `TInetSvrFactory` with `TBoundSvrFactory` if you want to create FastCGI application which run and managed by web server, for example Apache with `mod_fcgid` module. See
[fano-fcgid](https://github.com/fanoframework/fano-fcgid) example demo application.

`TInetSvrFactory`, `TUnixSvrFactory` and `TBoundSvrFactory` use [*select*](http://man7.org/linux/man-pages/man2/select.2.html) to monitor if socket is ready for I/O operation. To use [*epoll*](http://man7.org/linux/man-pages/man7/epoll.7.html), replace with `TEpollInetSvrFactory`, `TEpollUnixSvrFactory` and `TEpollBoundSvrFactory`. If you use FreeBSD, to use [*kqueue*](https://www.freebsd.org/cgi/man.cgi?kqueue), replace with `TKqueueInetSvrFactory`, `TKqueueUnixSvrFactory` and `TKqueueBoundSvrFactory`. You need to add conditional compilation define `$DEFINE USE_KQUEUE` or add `-dUSE_KQUEUE` line in `build.cfg` file. 

See [*Deploy as FastCGI application*](/deployment/fastcgi) for information how to
deploy FastCGI application on various web servers.

You may want to read [*Scaffolding FastCGI project directory structure*](/scaffolding-with-fano-cli/creating-project#scaffolding-fastcgi-project) for information how to scaffolding FastCGI web application with [Fano CLI](https://github.com/fanoframework/fano-cli).

## SCGI Application

To create web application that support [SCGI (Simple Common Gateway Interface)](https://python.ca/scgi/protocol.txt) protocol, create application service provider which use `TScgiAppServiceProvider` as shown in following code,

```
    appInstance := TDaemonWebApplication.create(
        TScgiAppServiceProvider.create(
            TServerAppServiceProvider.create(
                TAppServiceProvider.create(),
                (TInetSvrFactory.create(host, port) as ISocketSvrFactory).build()
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
                (TInetSvrFactory.create(host, port) as ISocketSvrFactory).build()
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

## Http Application

To create web application that support http protocol using libmicrohttpd, create application service provider which use `TMhdAppServiceProvider` also fill web server configuration as shown in following code,

```
var
    svrConfig : TMhdSvrConfig;
...
    svrConfig.host := 'example.fano';
    svrConfig.port := 8080;
    svrConfig.documentRoot := getCurrentDir() + '/public';
    svrConfig.serverName := 'example.fano';
    svrConfig.serverAdmin := 'admin@example.fano';
    svrConfig.serverSoftware := 'Fano Framework Web App';
    svrConfig.timeout := 120; //connection timeout in sec

    appInstance := TDaemonWebApplication.create(
        TMhdAppServiceProvider.create(
            TAppServiceProvider.create(),
            svrConfig
        ),
        TAppRoutes.create()
    );
```
See [fano-http](https://github.com/fanoframework/fano-http) example demo application.

See [Deploy as standalone web server](/deployment/standalone-web-server) for information how to
deploy http application as stand alone web server or through various reverse proxy web servers.

You may want to read
[Scaffolding libmicrohttpd project directory structure](/scaffolding-with-fano-cli/creating-project#scaffolding-libmicrohttpd-project) for information how to scaffolding libmicrohttpd web application easily.

## <a name="use-ipv6-address"></a>Use IPv6 address

If you want to create web application server which binds to [IPv6 address](https://tools.ietf.org/html/rfc4291), replace `TInetSvrFactory`, `TEpollInetSvrFactory`, `TKqueueInetSvrFactory` with `TInet6SvrFactory`, `TEpollInet6SvrFactory`, `TKqueueInet6SvrFactory` class respectively.

You may need to replace server address in proxy module of web server. For example for SCGI application using Apache `mod_proxy_scgi` module, you need to change virtual host configuration as shown in following example.

```
# IPv4 address
# ProxyPassMatch ^/(.*)$ "scgi://127.0.0.1:20477"

# IPv6 address
ProxyPassMatch ^/(.*)$ "scgi://[::1]:20477"
```
Please note that the IPv6 address needs to be inside square brackets.

See [fano-ipv6](https://github.com/fanoframework/fano-ipv6) example demo application which use IPv6.

For libmicrohttpd-based web application, you can add support to IPv6 address by setting
field `useIPv6` of `TMhdSvrConfig` record to `true`. If you need both IPv4 and IPv6, set field `dualStack` of `TMhdSvrConfig` to `true`.

```
var svrConfig : TMhdSvrConfig;
...
svrConfig.useIPv6 :=true;
svrConfig.dualStack := false; //IPv6 only
```
See [fano-mhdipv6](https://github.com/fanoframework/fano-mhdipv6) example demo http application which use IPv6.

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
- [Deploy standalone web server](/deployment/standalone-web-server)
