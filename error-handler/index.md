---
title: Error Handler
description: Tutorial on how to handle error with Fano Framework
---

<h1 class="major">Error Handler</h1>

## Handling exception

In web application built with Fano Framework, any unhandled exceptions that are raised will be handled by `IErrorHandler` interface instance.

This interface has one method

```
function handleError(
    const env : ICGIEnvironmentEnumerator;
    const exc : Exception;
    const status : integer = 500;
    const msg : string  = 'Internal Server Error'
) : IErrorHandler;
```

- `env` is environment variables enumerator. Read [CGI Environment](/environment) for more information about environment variables.
- `exc` is exception instance to be handled.
- `status` is integer value of HTTP code to send to client.
- `msg` is string value of HTTP message to send to client

If you take a look at Fano Framework sample applications, e.g.,
[Fano App](https://github.com/fanoframework/fano-app), you can observe
that error handler instance is injected during application instance creation.

The reason that error handler instance is injected directly in constructor and
not relying on dependency container is to ensure that when you create application
you must provide the error handler. It is mandatory. While when using
dependency container, developer may forget to inject error handler instance.

## Built-in error handler implementation

Fano Framework comes with several `IErrorHandler` implementation.

- `TFancyErrorHandler`, default basic error handler that output error in themed HTML. You use this mostly for development purpose as it displays verbose error information.
- `TErrorHandler`, basic error handler that output error in plain HTML. You use this mostly for development purpose as it displays verbose error information.
- `TAjaxErrorHandler`, error handler that output error informations as JSON.
- `TNullErrorHandler`, error handler that output nothing except HTTP error.
- `THtmlAjaxErrorHandler`, composite error handler that output basic HTML error or JSON based on whether request is AJAX or not.
- `TLogErrorHandler`, error handler that log error information instead of output it to client.
- `TTemplateErrorHandler`, error handler that output error using HTML template. This class is provided to enable developer to display nicely formatted error page. For production setup, this is mostly what you use.
- `TCompositeErrorHandler` error handler that is composed from two other error handler. This is provided so we combine, for example, log error to file and also displaying nicely formatted output to client.
- `TDecoratorErrorHandler` abstract error handler that is decorate other error handler.
- `TConditionalErrorHandler` abstract error handler that is select one from two error handlers based on a condition. Descendant must implement its `condition()` abstract method.
- `TBoolErrorHandler` error handler that is select one from two error handlers based on a condition specified in constructor parameter.
- `TNotFoundErrorHandler` error handler that is select one from two error handlers based on a condition if the the exception is `ERouteHandlerNotFound`.

## Display verbose error message

`TFancyErrorHandler` and `TErrorHandler` are basic implementation of `IErrorHandler`, which
display basic error information in HTML content type such as type of exception
raised, exception message and stacktrace and environment variables.

If you compile application with debugging information such as `-g` or `-gh` flag,
stacktrace is quite detail such as line number in source file. Without debugging information, it only displays memory location.

By default, when you inherit application service provider from `TBasicAppServiceProvider`, you use `TFancyErrorHandler` error handler.

```
type

    TAppServiceProvider = class(TBasicAppServiceProvider)
    public
        procedure register(const container : IDependencyContainer); override;
    end;
```

To register different error handler type, you need to override `buildErrorHandler()` method.

## Display verbose error message as JSON

```
type

    TAppServiceProvider = class(TBasicAppServiceProvider)
    protected
        function buildErrorHandler(
            const ctnr : IDependencyContainer;
            const config : IAppConfiguration
        ) : IErrorHandler; override;
        ...
    end;
...

    function TAppServiceProvider.buildErrorHandler(
        const ctnr : IDependencyContainer;
        const config : IAppConfiguration
    ) : IErrorHandler;
    begin
        result := TAjaxErrorHandler.create();
    end;
```

## Hide error message

```
function TAppServiceProvider.buildErrorHandler(
    const ctnr : IDependencyContainer;
    const config : IAppConfiguration
) : IErrorHandler;
begin
    result := TNullErrorHandler.create();
end;
```

## Display error with html template

```
function TAppServiceProvider.buildErrorHandler(
    const ctnr : IDependencyContainer;
    const config : IAppConfiguration
) : IErrorHandler;
begin
    result := TTemplateErrorHandler.create('/path/to/error/template.html');
end;
```

## Log error to file

```
function TAppServiceProvider.buildErrorHandler(
    const ctnr : IDependencyContainer;
    const config : IAppConfiguration
) : IErrorHandler;
begin
    result := TLogErrorHandler.create(TFileLogger.create('/path/to/log/file'));
end;
```

You need to make sure that `/path/to/log/file` is writeable.

## Log error to file and display error from template

```
function TAppServiceProvider.buildErrorHandler(
    const ctnr : IDependencyContainer;
    const config : IAppConfiguration
) : IErrorHandler;
begin
    result := TCompositeErrorHandler.create(
        TLogErrorHandler.create(TFileLogger.create('/path/to/log/file')),
        TTemplateErrorHandler.create('/path/to/error/template.html')
    );
end;
```

## Log error to file and display blank error page

```
function TAppServiceProvider.buildErrorHandler(
    const ctnr : IDependencyContainer;
    const config : IAppConfiguration
) : IErrorHandler;
begin
    result := TCompositeErrorHandler.create(
        TLogErrorHandler.create(TFileLogger.create('/path/to/log/file')),
        TNullErrorHandler.create()
    );
end;
```

## Select different error handler based on production or development configuration

```
function TAppServiceProvider.buildErrorHandler(
    const ctnr : IDependencyContainer;
    const config : IAppConfiguration
) : IErrorHandler;
var prodErrHandler : IErrorHandler;
    devErrHandler : IErrorHandler;
    isProd : boolean;
begin
    prodErrHandler := TCompositeErrorHandler.create(
        TLogErrorHandler.create(TFileLogger.create('/path/to/log/file')),
        TTemplateErrorHandler.create('/path/to/error/template.html')
    );
    devErrHandler := TErrorHandler.create();
    isProd := config.getBool('isProduction');
    result := TBoolErrorHandler.create(
        prodErrHandler,
        devErrHandler,
        isProd
    );
end;
```

## Create your own error handler

If you find built-in error handlers are not enough for your use case, you may create your own `IErrorHandler` implementation.

```
unit myerrorhandler;

interface

{$MODE OBJFPC}
{$H+}

uses

    fano;

type

    TMyErrorHandler = class(TInterfacedObject, IErrorHandler)
    public
        function handleError(
            const env : ICGIEnvironmentEnumerator;
            const exc : Exception;
            const status : integer = 500;
            const msg : string  = 'Internal Server Error'
        ) : IErrorHandler;
    end;

implementation

    function TMyErrorHandler.handleError(
        const env : ICGIEnvironmentEnumerator;
        const exc : Exception;
        const status : integer = 500;
        const msg : string  = 'Internal Server Error'
    ) : IErrorHandler;
    begin
        writeln('Content-Type: text/html');
        writeln('Status: ', status, ' ', msg) ;
        writeln();
        writeln('Funky error message');
        result := self;
    end;

end.
```

Following lines are mandatory to conform with CGI protocol.

```
writeln('Content-Type: text/html');
writeln('Status: ', status, ' ', msg) ;
writeln();
```

To use your own error handler

```
uses

    fano,
    myerrorhandler;

function TAppServiceProvider.buildErrorHandler(
    const ctnr : IDependencyContainer;
    const config : IAppConfiguration
) : IErrorHandler;
begin
    result := TMyErrorHandler.create();
end;
```

## Explore more

- [Working with Views](/working-with-views)
