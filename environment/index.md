---
title: CGI Environment
description: Tutorial on how to work with CGI Environment
---

<h1 class="major">CGI Environment</h1>

## About environment

When our application run as CGI, FastCGI or SCGI application, it will receive environment variables from web server. CGI Environment variables are part of
CGI protocol. [See RFC 3875](https://tools.ietf.org/html/rfc3875) for more information about CGI environment variables.

In Fano Framework, environment variable is encapsulated in `ICGIEnvironment` interface.

You can get `ICGIEnvironment` interface instance inside controller from `IRequest` interface instance through property `env` or method `getEnvironment()`.

```
...
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse
) : IResponse;
var myEnv : ICGIEnvironment;
begin
    myEnv := request.env;
    ...
end;
```

Read [Working with Controllers](/working-with-controllers) for more information about `handleRequest()` method.

## ICGIEnvironment interface

`ICGIEnvironment` interface provides several methods for reading CGI environment variables but most important is `env()` method. Other methods basically just
act as helper method which actually use `env()`  under the hood. Look at [ICGIEnvironment source](https://github.com/fanoframework/fano/blob/master/src/Environment/Contracts/EnvironmentIntf.pas) for more information on available methods.

```
function env(const keyName : string) : string;
```

It retrieves single environment variable value using its name.

## Retrieving environment variables

### Get client IP address

```
var myEnv : ICGIEnvironment;
    ipAddr : string;
...
ipAddr := myEnv.env('REMOTE_ADDR');
```

or

```
ipAddr := myEnv.remoteAddr();
```

### Get host

```
var myEnv : ICGIEnvironment;
    ipAddr : string;
...
host := myEnv.env('HTTP_HOST');
```

or

```
host := myEnv.httpHost();
```

### Get request scheme

```
var myEnv : ICGIEnvironment;
    scheme : string;
...
scheme := myEnv.env('REQUEST_SCHEME');
```

or

```
scheme := myEnv.requestScheme();
```
For example, `scheme` will contains `https` if your application is invoked via HTTPS protocol.

### Get request method

```
var myEnv : ICGIEnvironment;
    method : string;
...
method := myEnv.env('REQUEST_METHOD');
```

or

```
method := myEnv.requestMethod();
```
For example, `method` will contains `GET` if your application is invoked via HTTP GET.

### Get query string

```
var myEnv : ICGIEnvironment;
    query : string;
...
query := myEnv.env('QUERY_STRING');
```

or

```
query := myEnv.queryString();
```

### Get user agent

```
var myEnv : ICGIEnvironment;
    browser : string;
...
browser := myEnv.env('HTTP_USER_AGENT');
```

or

```
browser := myEnv.httpUserAgent();
```

## Built-in ICGIEnvironment implementation

Fano Framework provides several built-in implementation of ICGIEnvironment.

- `TCGIEnvironment` class which get environment variables thorough `GetEnvironmentVariable()` function from FreePascal `SysUtils` unit. For CGI application, this is what you use.
- `TKeyValueEnvironment` class which get environment variables thorough key value pair. For FastCGI orr SCGI application, this is what you use.

## Enumerate all environment variables

Sometime you want to know all current environment variables that web server sends to application. For example `TErrorHandler` class needs to be able to dump all environment variables. See [TErrorHandler source](https://github.com/fanoframework/fano/blob/master/src/Error/ErrorHandlerImpl.pas).

To be able to enumerate all available variables, you need to use `ICGIEnvironmentEnumerator` interface. You get this interface instance from `ICGIEnvironment` interface through its `enumerator` property or `getEnumerator()` method.

Environment enumerator provides three methods which enables developer to iterate all environment variables.

- `count()` returns number of environment variables.
- `getKey()` return environment variable name using zero-based index.
- `getValue()` return environment variable value using zero-based index.

Read [ICGIEnvironmentEnumerator source](https://github.com/fanoframework/fano/blob/master/src/Environment/Contracts/EnvironmentEnumeratorIntf.pas) for more information.

```
var myEnv : ICGIEnvironment;
    envEnum : ICGIEnvironmentEnumerator;
    indx : integer;
    varName, varValue : string;
...
envEnum := myEnv.enumerator;
for indx := 0 to envEnum.count-1 do
begin
    varName := envEnum.getKey(indx);
    varValue := envEnum.getValue(indx);
end;

```

## Explore more

- [Dispatcher](/dispatcher)
- [Error Handler](/error-handler)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
