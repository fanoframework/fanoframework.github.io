---
title: Error Handler
description: Tutorial on how to handle error with Fano Framework
---

<h1 class="major">Error Handler</h1>

Any application will face situation where something wrong happen and we can not provide solution other than tell user about it. For example, user tries to retrieve nonexistent resource.

<img src="/assets/images/404.jpg" alt="Error illustration" width="100%">

Photo by <a href="https://unsplash.com/@woodies11?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Romson Preechawit</a> on <a href="https://unsplash.com/s/photos/error?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

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

Error handler are registered in application service provider `buildErrorHandler()` method [as shown in example code below](#display-verbose-error-message-as-json).

## Exceptions

Fano Framework defines some exception classes which corresponds to [HTTP error code](https://datatracker.ietf.org/doc/html/rfc7231).

- `EBadRequest`, HTTP 400 Bad Request.
- `EUnauthorized`, HTTP 401 Unauthorized.
- `EForbidden`, HTTP 403 Forbidden
- `ENotFound`, HTTP 404 Not Found.
- `EMethodNotAllowed`, HTTP 405 Method Not Allowed.
- `ENotAcceptable`, HTTP 406 Not Acceptable.
- `EGone`, HTTP 410 Gone.
- `EUnsupportedMediaType`, HTTP 415 Unsupported Media Type.
- `EUnprocessableEntity`, HTTP 422 Unprocessable Entity.
- `ETooManyRequest`, HTTP 429 Too Many Request.
- `EInternalServerError`, HTTP 500 Internal Server Error.
- `ENotImplemented`, HTTP 501 Not Implemented.
- `EBadGateway`, HTTP 502 Bad Gateway.
- `EServiceUnavailable`, HTTP 503 Service Unavailable.
- `EGatewayTimeout`, HTTP 504 Gateway Timeout.

If you raise any of exception above, Fano Framework returns its corresponding HTTP error as response. All exception classes above are derived from `EHttpException` class.

Fano Framework also defines other exceptions not related to HTTP error code. Any of these exceptions will result in HTTP 500 error response except `ERouteHandlerNotFound` which result in HTTP 404 error.

## Built-in error handler implementation

Fano Framework comes with several `IErrorHandler` implementation.

- `TFancyErrorHandler`, default basic error handler that output error in themed HTML. You use this mostly for development purpose as it displays verbose error information.
- `TErrorHandler`, basic error handler that output error in plain HTML. You use this mostly for development purpose as it displays verbose error information.
- `TAjaxErrorHandler`, error handler that output error informations as JSON.
- `TNullErrorHandler`, error handler that output nothing except HTTP error.
- `THtmlAjaxErrorHandler`, composite error handler that output basic HTML error or JSON based on whether request is AJAX or not.
- `TLogErrorHandler`, error handler that log error information instead of output it to client.
- `TLoggerErrorHandler`, error handler that similar to `TLogErrorHandler` except it can determine what log level to be log. For example it can log critical type log only.
- `TTemplateErrorHandler`, error handler that output error using HTML template. This class is provided to enable developer to display nicely formatted error page. For production setup, this is mostly what you use.
- `TCompositeErrorHandler` error handler that is composed from two other error handler. This is provided so we combine, for example, log error to file and also displaying nicely formatted output to client. To combine three or more error handlers, you need to daisy-chain them.
- `TGroupErrorHandler` error handler that is composed from one or more error handlers. This is similar to composite error handler above, except, it is more flexible as you can compose arbitrary number of error handlers.
- `TDecoratorErrorHandler` abstract error handler that is decorate other error handler.
- `TConditionalErrorHandler` abstract error handler that is select one from two error handlers based on a condition. Descendant must implement its `condition()` abstract method.
- `TBoolErrorHandler` error handler that is select one from two error handlers based on a condition specified in constructor parameter.
- `TNotFoundErrorHandler` error handler that is select one from two error handlers based on a condition if the the exception is `ERouteHandlerNotFound`. This is provided so you can handle HTTP 404 error separately, for example, to display different HTML template for HTTP 404 error.
- `TMethodNotAllowedErrorHandler` error handler that is select one from two error handlers based on a condition if the the exception is `EMethodNotAllowed`. This is provided so you can handle HTTP 405 error separately, for example, to use different HTML template for HTTP 405 error.
- `TInternalServerErrorHandler` error handler that is select one from two error handlers based on a condition if the the exception is `EInternalServerError`. This is provided so you can handle HTTP 500 error separately, for example, to use different HTML template for HTTP 500 error.

## <a name="display-verbose-error-message"></a>Display verbose error message

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

## <a name="display-verbose-error-message-as-json">Display verbose error message as JSON

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

Or if you want to log emergency and critical message only, use `TLoggerErrorHandler` instead.
```
function TAppServiceProvider.buildErrorHandler(
    const ctnr : IDependencyContainer;
    const config : IAppConfiguration
) : IErrorHandler;
begin
    result := TLoggerErrorHandler.create(
        TFileLogger.create('/path/to/log/file'),
        [ logEmergency, logCritical ]
    );
end;
```

Read [Using Logger documentation](/utilities/using-loggers) for more information on how to use logger utility.

## Log error to file and display error from template

Use `TCompositeErrorHandler` or `TGroupErrorHandler` to compose several error handlers as one.

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
or with `TGroupErrorHandler`
```
function TAppServiceProvider.buildErrorHandler(
    const ctnr : IDependencyContainer;
    const config : IAppConfiguration
) : IErrorHandler;
begin
    result := TGroupErrorHandler.create([
        TLogErrorHandler.create(TFileLogger.create('/path/to/log/file')),
        TTemplateErrorHandler.create('/path/to/error/template.html')
    ]);
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

## Compose several error handlers

Use `TCompositeErrorHandler` with daisy-chain or `TGroupErrorHandler` to compose several error handlers as one.

```
function TAppServiceProvider.buildErrorHandler(
    const ctnr : IDependencyContainer;
    const config : IAppConfiguration
) : IErrorHandler;
begin
    result := TCompositeErrorHandler.create(
        TCompositeErrorHandler.create(
            //log to file and STDERR
            TLogErrorHandler.create(TFileLogger.create('/path/to/log/file')),
            TLogErrorHandler.create(TStdErrLogger.create())
        ),
        TTemplateErrorHandler.create('/path/to/error/template.html')
    );
end;
```
or with `TGroupErrorHandler`
```
function TAppServiceProvider.buildErrorHandler(
    const ctnr : IDependencyContainer;
    const config : IAppConfiguration
) : IErrorHandler;
begin
    //log error to file and STDERR and also display error template
    result := TGroupErrorHandler.create([
        TLogErrorHandler.create(TFileLogger.create('/path/to/log/file')),
        TLogErrorHandler.create(TStdErrLogger.create()),
        TTemplateErrorHandler.create('/path/to/error/template.html')
    ]);
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
- [Using Loggers](/utilities/using-loggers)
