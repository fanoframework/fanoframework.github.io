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
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
var respBody : IResponseStream;
begin
   respBody := response.body();
   respBody.write(
        '<html>' +
        '<head><title>Hello world</title></head>' +
        '<body><p>Hello world</p></body>' +
        '/<html>'
    )
    result := response;
end;
```

Should you need, content of stream as string, you can call `read()` method.

```
var response : IResponse;
    resp : string;
...
resp := response.body().read();
```

To output content of stream including response header, you can call `write()` method.
All built-in implementations of `IResponse` interface just output to STDOUT.

```
var response : IResponse;
...
response.write();
```

## Setting response header

`IResponse` provides collection of response header through `header()` method.
It returns instance of `IHeaders` interface.

- `setHeader()` method set value of a HTTP response header.
- `addHeader()` method add value of a HTTP response header.
- `getHeader()` method get value of a HTTP response header.
- `has()` method test if HTTP response header is set.
- `writeHeaders()` method output all headers.

For example, to set `Content-Type` header with value of `application/json`,

```
var response : IResponse;
...
response.headers().setHeader('Content-Type', 'application/json');
```

If header is already set, calling `setHeader()` will overwrite previous value. If you want to add new header and not overwriting previous value, use `addHeader()`.

```
response.headers().addHeader('Content-Type', 'application/json');
```

## Built-in IResponse implementation

Fano Framework comes with several built-in `IResponse` implementations to simplify task regarding response.

- `TResponse`, this class is mostly what you get from dispatcher when controller is invoked.
- `TJSONResponse`. This is response that you may use to output JSON format. It set `Content-Type` header to `application/json`.
- `TBinaryResponse`.This is response that you may need to send binary data to browser, such as image response. See [Fano App Image](https://github.com/fanoframework/fano-app-img) demo application to see how to return binary response.
- `TRedirectResponse` is response for doing HTTP redirection. Read [Redirection](#redirection-response) section on this document.
- `THttpCodeResponse` is response for setting up HTTP status manually. Read [Response with HTTP Status Code](#response-with-status-code) for more information.

## <a name="redirection-response"></a>Redirection Response

To simplify send redirection response, Fano Framework provides `TRedirectResponse`.

For example, in controller

```
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
var headers : IHeaders;
begin
    //redirect to fanoframework.github.io
    result := TRedirectResponse.create(
        response.headers(),
        'https://fanoframework.github.io'
    );
end;
```

By default, above code will redirect browser to [https://fanoframework.github.io](https://fanoframework.github.io) with HTTP 302 status. If you need to use different HTTP status code, set it in constructor's third parameter

```
//redirect to fanoframework.github.io with HTTP 304
result := TRedirectResponse.create(
    response.headers(),
    'https://fanoframework.github.io',
    304
);
```

## JSON response

To simplify output JSON response, you can use `TJsonResponse` as shown in following code

```
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
begin
    result := TJsonResponse.create(
        response.headers(),
        '{ "foo" : "bar" }'
    );
end;
```

## Binary response

To simplify to output binary response such as image, you can use `TBinaryResponse` as shown in following code,

```
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
var mem : TStream;
begin
    ...
    result := TBinaryResponse.create(
        response.headers(),
        'image/png',
        TResponseStream.create(mem)
    );
end;
```

In the code above `mem` is assume to contain PNG file binary data. `TResponseStream` is adapter class that implements `IResponseStream` interface to be able to read `TStream` as string.

To be able to display other binary format, just pass correct `Content-Type` header. For example, to output PDF document

```
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
var mem : TStream;
    pdf : TPDFDocument;
begin
    //build pdf document and create stream
    ...
    pdf.saveToStream(mem);
    mem.seek(0, soFromBeginning);
    result := TBinaryResponse.create(
        response.headers(),
        'application/pdf',
        TResponseStream.create(mem)
    );
end;
```

Please note that `TPDFDocument` is part of Free Pascal `fcl-pdf` library.

## <a name="response-with-status-code"></a>Response with HTTP Status Code

To return response with a HTTP status code, you can either calling `setHeader()` as shown in following code,

```
response.headers().setHeader('Status', '400 Bad Request');
```

or you can use `THttpCodeResponse`,

```
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
begin
    result := THttpCodeResponse.create(
        400,
        'Bad Request',
        response.headers().clone() as IHeaders
    );
end;
```

## Explore more

- [Working with Request](/working-with-request)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
