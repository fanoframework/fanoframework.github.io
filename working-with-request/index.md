---
title: Working with Request
description: Tutorial on how to work with Request object in Fano Framework
---

<h1 class="major">Working with Request</h1>

## About Request

When route handler (or controller) is executed, [dispatcher](/dispatcher) will
call its `handleRequest()` and pass request object that contains handful data
related to current request.

This request object is instance of `IRequest` interface that can be queried for
data such as query strings, cookies, POST data send by client or uploaded files.

Read [Working with Controllers](/working-with-controllers) for more information on hwo to work with route handler  or controller.

## Getting query parameters

If you registered route `/home` and user open `http://[your hostname]/home?msg1=test&msg2=nice`

To retrieve value of `msg1` and `msg2` parameters, you can use `getQueryParam()`
method.

```
var msg1, msg2 : string;
...
msg1 := request.getQueryParam('msg1'); //msg1 = 'test'
msg2 := request.getQueryParam('msg2'); //msg2 = 'nice'
```

To set default value if `msg1` is not set,

```
msg1 := request.getQueryParam('msg1', 'ok');
```

To retrieve all query parameters at once, use `getQueryParams()`

```
var params : IReadOnlyList;
...
params := request.getQueryParams();
```

## Retrieve cookies

To retrieve value of `msg1` and `msg2` parameters from cookies, you can use `getCookieParam()` method.

```
var msg1, msg2 : string;
...
msg1 := request.getCookieParam('msg1');
msg2 := request.getCookieParam('msg2');
```

To set default value if `msg1` cookie is not set,

```
msg1 := request.getCookieParam('msg1', 'ok');
```

To retrieve all cookie parameters at once, use `getCookieParams()`

```
var cookies : IReadOnlyList;
...
cookies := request.getCookieParams();
```

## Get POST data

If you have form like following snippet

```
<form action="/submit" method="post">
    <input type="text" name="msg1" value="test">
    <input type="text" name="msg2" value="nice">
    <button type="submit">Submit</button>
</form>
```

To retrieve value of `msg1` and `msg2` parameters from POST data, you can use `getParsedBodyParam()` method.

```
var msg1, msg2 : string;
...
msg1 := request.getParsedBodyParam('msg1');
msg2 := request.getParsedBodyParam('msg2');
```

To set default value if `msg1` parameter is not set,

```
msg1 := request.getParsedBodyParam('msg1', 'ok');
```

To retrieve all POST parameters at once, use `getParsedBodyParams()`

```
var params : IReadOnlyList;
...
params := request.getParsedBodyParams();
```

## Get query parameters and POST data

To get query string parameters and POST data as one list

```
var params : IReadOnlyList;
...
params := request.getParams();
```

To read single value

```
msg1 := request.getParam('msg1', 'ok');
```

Above method search query string parameter first. If it cannot find data with key `msg1`, it tries to read body parameters. If none found, default value 'ok` is returned.

## Read request header

To get request headers, use `headers()` methods of `IRequest`. It returns instance of `IReadOnlyHeaders` interface.

To read request header value, call `getHeader()` method and pass header to read. For example to read `Accept-Language` header value,

```
var acceptLang : string;
...
acceptLang := request.headers().getHeader('Accept-Language');
```

To test availability of header, call `has()` method. For example to test if `Accept-Language` header is set,

```
var langAvail : boolean;
...
langAvail := request.headers().has('Accept-Language');
```

## Explore more

- [Working with response](/working-with-response)
- [Working with Controllers](/working-with-controllers)
- [Working with Views](/working-with-views)
- [Form Validation](/security/form-validation)
- [Handling CORS](/security/handling-cors)
- [Handling File Upload](/handling-file-upload)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
