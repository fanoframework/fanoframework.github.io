---
title: Dispatcher
description: Request dispatcher in Fano Framework
---

<h1 class="major">Dispatcher</h1>

## Dispatcher task

In general, task of a dispatcher is quite simple. Given request uri, it must find instance of class that will handle the request, passing all relevant informations to it, execute the handler and return response.

A dispatcher in Fano Framework must implements `IDispatcher` interface.

```
IDispatcher = interface
    ['{F13A78C0-3A00-4E19-8C84-B6A7A77A3B25}']
    function dispatchRequest(const env: ICGIEnvironment) : IResponse;
end;
```
- `env`, CGI environment variable that is given by web server.


