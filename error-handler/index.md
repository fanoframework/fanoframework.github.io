---
title: Error Handler
description: Tutorial on how to handle error with Fano Framework
---

<h1 class="major">Error Handler</h1>

## Handling exception

In web application built with Fano Framework, exception that is raised will be handled by `IErrorHandler` interface instance.

This interface has one method

```
(*!---------------------------------------------------
 * handle exception
 *----------------------------------------------------
 * @param exc exception that is to be handled
 * @param status HTTP error status, default is HTTP error 500
 * @param msg HTTP error message
 * @return current instance
 *---------------------------------------------------*)
function handleError(
    const exc : Exception;
    const status : integer = 500;
    const msg : string  = 'Internal Server Error'
) : IErrorHandler;
```

If you take a look at Fano Framework sample applications, e.g.,
[Fano App](https://github.com/fanoframework/fano-app), you can observe
that error handler instance is injected during application instance creation.

The reason that error handler instance is injected directly in constructor and
not relying on dependency container is to ensure that when you create application
you must provide the error handler. It is mandatory. While when using
dependency container, developer may forget to inject error handler instance.

```
var appInstance : IWebApplication;
...
appInstance := TMyApp.create(
    TDependencyContainer.create(TDependencyList.create()),
    TCGIEnvironment.create(),
    TErrorHandler.create()
);
```

`TErrorHandler` is one of basic implementation of `IErrorHandler`, which
display basic error information in HTML content type such as type of exception
raised, exception message and stacktrace.

## Built-in error handler implementation

Fano Framework comes with several `IErrorHandler` implementation.

- `TErrorHandler`, basic error handler that output error in plain HTML.
- `TAJaxErrorHandler`, error handler that output error informations as JSON.
- `TNullErrorHandler`, error handler that output nothing except HTTP error.
- `THtmlAjaxErrorHandler`, composite error handler that output basic HTML error or JSON based on whether request is AJAX or not.
- `TLogErrorHandler`, error handler that log error information instead of output it to client.
- `TTemplateErrorHandler`, error handler that output error using HTML template. This class is provided to enable developer to display nicely formatted error page.
- `TCompositeErrorHandler` error handler that is composed from two other error handler. This is provided so we combine, for example, log error to file and also displaying nicely formatter output to client