---
title: Working with Response
description: Tutorial on how to work with Response object in Fano Framework
---

<h1 class="major">Working with Response</h1>

## About Response

When route handler (or controller) is executed, [dispatcher](/dispatcher) will
call its `handleRequest()` and pass response object which instance of `IResponse`interface. For information about controller, read [Working with Controller](/working-with-controllers).

Your route handler or controller is required to return response object. You can either return response object given by dispatcher object or create new one.

`IResponse` is mostly used in conjunction with view. Read [Working with Views](/working-with-views) to learn more about view in Fano Framework.

## Writing response body

`IResponse` interface has `body()` method which will return `IResponseStream`
interface instance. `IResponseStream` is descendant of `IStreamAdapter` interface
which add additional methods to simplify writing string to stream and also reading string from it.

```
var response : IResponse;
    respBody : IResponseStream;
...
respBody := response.body();
respBody.write(
    '<html>' +
    '<head><title>Hello world</title></head>' +
    '<body><p>Hello world</p></body>' +
    '/<html>'
);
```

Should you need, content of stream as string, you can call `read()` method.

```
var response : IResponse;
    resp : string;
...
resp := response.body().read();
```

To output content of stream including response header, you can call `write()` method.
All built-in implementations of `IResponse` interface are just outpur to STDOUT.

```
var response : IResponse;
...
response.write();
```

## Setting response header

`IResponse` provides collection of response header through `header()` method.
It returns instance of `IHeaders` interface.

- `setHeader()` method set value of a HTTP response header.
- `getHeader()` method get value of a HTTP response header.
- `has()` method test if HTTP response header is set.
- `writeHeaders()` method output all headers.

For example, to set `Content-Type` header with value of `application/json`,

```
var response : IResponse;
...
response.headers().setHeader('Content-Type', 'application/json');
```

If header is already set, calling `setHeader()` will overwrite previous value.

## Built-in IResponse implementation

Fano Framework comes with several built-in `IResponse` implementations to simplify task regarding response.

- `TResponse`, this class is mostly what you get from dispatcher when controller is invoked.
- `TJSONResponse`. This is response that you may use to output JSON format. It set `Content-Type` header to `application/json`.
- `TBinaryResponse`.This is response that you may need to send binary data to browser, such as image response. See [Fano App Image](https://github.com/fanoframework/fano-app-img) demo application to see how to return binary response.
- `TRedirectResponse` is response for doing HTTP redirection. See *Redirection* section on this document.

## Redirection Response

To simplify send redirection response, Fano Framework provides `TRedirectResponse`.

For example, in controller

```
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse
) : IResponse;
vr headers : IHeaders;
begin
    //create new copy of current response header
    headers := response.headers().clone();

    //redirect to fanoframework.github.io
    result := TRedirectResponse.create(headers, 'https://fanoframework.github.io');
end;
```
