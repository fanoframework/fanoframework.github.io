---
title: HTTP Verb Tunnelling
description: Handling HTTP werb tunnelling issue in Fano Framework
---

<h1 class="major">HTTP Verb Tunnelling</h1>

## What is it?

HTTP verb tunnelling (sometime called HTTP method override) is actually hack. This is provided to solve situation where web application running behind strict-policy firewall which only allows GET and POST request.

To allow PUT, DELETE, OPTIONS, PATCH method to be used, developer can use POST method and `X-Http-Method-Override` request header set to appropriate method as shown in following snippet,

```
POST /delete HTTP/1.1
Host: myapp.fano
Content-type: application/json
Content-length: 16
X-Http-Method-Override: DELETE

{"data":"value"}
```

Alternatively, you can also use special request body parameter `_method` like so.

```
POST /delete HTTP/1.1
Host: myapp.fano
Content-type: application/x-www-form-urlencoded
Content-length: 25

data=value&_method=DELETE
```

In example above, `http://myapp.fano/delete` is assumed only handle DELETE request. Using apropriate header value, client can send as POST request which will be translated as DELETE request.

## Using HTTP verb tunnelling in Fano Framework

To allow HTTP verb tunnelling, you need to use `TXDispatcher` or `TXSimpleDispatcher` class and `TVerbTunnellingRequestResponseFactory` as shown in following example code,

```
function TAppServiceProvider.buildDispatcher(
    const ctnr : IDependencyContainer;
    const routeMatcher : IRouteMatcher;
    const config : IAppConfiguration
) : IDispatcher;
begin
    ctnr.add(
        GuidToString(IDispatcher),
        TXSimpleDispatcherFactory.create(
            ctnr[GuidToString(IRouter)] as IRouteMatcher,
            TVerbTunnellingRequestResponseFactory.create(
                TRequestResponseFactory.create()
            )
        )
    );
    result := ctnr[GuidToString(IDispatcher)] as IDispatcher;
end;
```

Using this type of dispatcher, when Fano Framework receives POST request with
`X-Http-Method-Override` header set or request parameter `_method`, it uses verb set in header or parameter value instead.
This new verb is tested against known HTTP verb (GET, POST, PUT, DELETE, PATCH, OPTIONS, HEAD).
If not one of allowed methods, `EInvalidMethod` exception is raised.

If you need to use this, you need to understand security implication of [HTTP verb tampering issue](https://www.owasp.org/index.php/Testing_for_HTTP_Verb_Tampering_(OTG-INPVAL-003)).

## Explore more

- [Dispatcher](/dispatcher)
- [Security](/security)
- [HTTP verb tunnelling example application](https://github.com/fanoframework/fano-verb-tunneling)
