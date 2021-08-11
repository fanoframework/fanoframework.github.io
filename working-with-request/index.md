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

Read [Working with Controllers](/working-with-controllers) for more information on how to work with route handler  or controller.

## <a name="getting-query-parameters"></a>Getting query parameters

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

## <a name="retrieve-cookies"></a>Retrieve cookies

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

## <a name="get-post-put-patch-data">Get POST/PUT/PATCH data

If you have form like following snippet

```
<form action="/submit" method="post">
    <input type="text" name="msg1" value="test">
    <input type="text" name="msg2" value="nice">
    <button type="submit">Submit</button>
</form>
```

To retrieve value of `msg1` and `msg2` parameters from POST/PUT/PATCH data, you can use `getParsedBodyParam()` method.

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

## Get query parameters and POST/PUT/PATCH data

To get query string parameters and POST/PUT/PATCH data as one list

```
var params : IReadOnlyList;
...
params := request.getParams();
```

To read single value

```
msg1 := request.getParam('msg1', 'ok');
```

Above method search query string parameter first. If it cannot find data with key `msg1`, it tries to read body parameters. If none found, default value `ok` is returned.

## <a name="read-request-header"></a>Read request header

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

## Get HTTP method

To get request http method such as GET, POST, etc., use `method` property.

```
var httpMethod : string;
...
httpMethod := request.method;
```

## Get request URI

To get request URI, use `uri()` method. It returns `IUri` interface which you can use to get information regarding current request URI such as scheme, host, port, path, query string, fragment. Read [RFC 3986](https://tools.ietf.org/html/rfc3986) for information regarding URI.

```
var reqUri : IUri;
...
reqUri := request.uri();
//if we have https://example.com:8080/hello?q=good#fano

//print uri scheme i.e, https
writeln(reqUri.scheme);

//print uri host i.e, example.com
writeln(reqUri.host);

//print uri port i.e, 8080
writeln(reqUri.port);

//print uri path i.e, /hello
writeln(reqUri.path);

//print uri query i.e, q=good
writeln(reqUri.query);

//print uri fragment i.e, fano
writeln(reqUri.fragment);
```

## Get current environment variables
You can get current CGI environment using `env` property.

```
var reqEnv : ICGIEnvironment;
...
reqEnv := request.env;
```
Read [CGI Environment](/environment) for more information.

## Check if request is AJAX

To test if request is AJAX request, use `isXhr()` method.

```
var ajax : boolean;
...
ajax := request.isXhr();
```

## <a name="handling-request-with-json-body"></a>Handling request with JSON body

To handle request with `application/json` body, such as following shell command

```
$ curl --header "Content-Type: application/json" \
    --request POST \
    --data '{"username":"xyz","password":"xyz"}' \
    http://yourapp.fano/submit
```

Fano Framework provides `TJsonRequest` class which decorates `IRequest` instance to handle such request. For example, to read `username` in request above,

```
var originalRequest, jsonRequest : IRequest;
...
jsonRequest := TJsonRequest.create(originalRequest);
username := jsonRequest.getParsedBodyParam('username');
password := jsonRequest.getParsedBodyParam('password');
```

For more complex JSON structure, you can read it using its name separated by dot. For JSON data,

```
{
    "username" : "xyz",
    "password" : "xyz",
    "login" : {
        "lastLogin" : "2020-02-10 10:10:00"
    }
}
```
You can read `lastLogin` data as follows,

```
lastLogin := jsonRequest.getParsedBodyParam('login.lastLogin');
```

The process of wrapping original request as `TJsonRequest` is implemented in `TJsonContentTypeMiddleware` middleware ([View TJsonContentTypeMiddleware source](https://github.com/fanoframework/fano/blob/master/src/Middleware/BuiltIns/JsonContentTypeMiddlewareImpl.pas)).
. When you attach this middleware to a route, everytime application receive method `POST`, `PUT`, `PATCH` or `DELETE` with `Content-Type` header set to `application/json`, original request will be wrapped inside `TJsonRequest` ([View TJsonRequest source](https://github.com/fanoframework/fano/blob/master/src/Http/Request/JsonRequestImpl.pas)).

To register this middleware, use `TJsonContentTypeMiddlewareFactory` class.

```
container.add(
    'jsonReq',
    TJsonContentTypeMiddlewareFactory.create()
);
```

In route registration, attach middleware to route, as shown in following code.

```
router.post(
    '/submit',
    container['submit.controller'] as IRequestHandler
).add(container['jsonReq'] as IMiddleware);
```

Please read [middleware documentation](/middlewares) for information how to work with middleware.

See [Fano Json Request](https://github.com/fanoframework/fano-json-request) for example how to use `TJsonContentTypeMiddleware` middleware to handle request with JSON body.

## Explore more

- [Working with response](/working-with-response)
- [Working with Controllers](/working-with-controllers)
- [Working with Views](/working-with-views)
- [Form Validation](/security/form-validation)
- [Handling CORS](/security/handling-cors)
- [Handling File Upload](/handling-file-upload)
