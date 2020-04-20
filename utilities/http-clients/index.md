---
title: Use HTTP client to call API
description: Tutorial on how to use HTTP client utilities provided by Fano Framework
---

<h1 class="major">HTTP Client</h1>

Fano Framework provides optional thin wrappers of HTTP clients such as curl and `TFPHttpClient` class that come with Free Pascal.

## HTTP client interfaces

Fano Framework wraps http client and segregate it based on HTTP method. List below shows all HTTP interfaces. These segregated interfaces are provided so developer have more control of what HTTP method is allowed to be executed in code.

### IHttpGetClient

Send HTTP GET request. It has one method `get()` that accept `url` of target and optional `data` parameter which contains any data associated with this request.

```
function get(
    const url : string;
    const data : ISerializeable = nil
) : IResponseStream;
```

This method returns response from web server as stream.

```
var http : IHttpGetClient;
    response : IResponseStream;
    resStr : string;
...
response := http.get('http://api.examples.com');
resStr := response.read();
```

### IHttpPostClient

Similar to `IHttpGetClient` but for HTTP POST request. It has one method `post()` with same parameters and same return value as `get()` method above.

```
var http : IHttpPostClient;
    postData : ISerializeable;
    response : IResponseStream;
    resStr : string;
...
response := http.post('http://api.examples.com', postData);
resStr := response.read();
```

### IHttpPutClient

Similar to `IHttpPostClient` but for HTTP PUT request.

```
var http : IHttpPutClient;
    putData : ISerializeable;
    response : IResponseStream;
    resStr : string;
...
response := http.put('http://api.examples.com', putData);
resStr := response.read();
```


### IHttpDeleteClient

Similar to `IHttpPostClient` but for HTTP DELETE request.

```
var http : IHttpDeleteClient;
    deleteData : ISerializeable;
    response : IResponseStream;
    resStr : string;
...
response := http.delete('http://api.examples.com', deleteData);
resStr := response.read();
```

### IHttpPatchClient
Similar to `IHttpPostClient` but for HTTP PATCH request.

```
var http : IHttpPatchClient;
    patchData : ISerializeable;
    response : IResponseStream;
    resStr : string;
...
response := http.patch('http://api.examples.com', patchData);
resStr := response.read();
```

### IHttpOptionsClient
Similar to `IHttpOptionsClient` but for HTTP OPTIONS request.

```
var http : IHttpOptionsClient;
    optData : ISerializeable;
    response : IResponseStream;
    resStr : string;
...
response := http.options('http://api.examples.com', optData);
resStr := response.read();
```

### IHttpHeadClient
Similar to `IHttpGetClient` but for HTTP HEAD request.

```
var http : IHttpHeadClient;
    response : IResponseStream;
    resStr : string;
...
response := http.head('http://api.examples.com');
resStr := response.read();
```

### IHttpClientHeaders

This is interface for any class having capability to set HTTP request headers. It contains two methods `add()` and `apply()` to add headers and apply headers respectively. Both method returns current instance.
```
function add(const aheader : string; const avalue : string) : IHttpClientHeaders;

function apply() : IHttpClientHeaders;
```
For example

```
var headers : IHttpClientHeaders;
...
headers.add('Content-Type', 'application/json');
headers.add('Content-Length', 120);
headers.apply();
```
## Initialize data for request
You can use any instance of objects so long it implements `ISerializeable` interface. For example,

```
var data : IModelParams;
    http : IHttpPostClient;
    headers : IHttpClientHeaders;
...
data := TModelParams.create();
data.writeString('firstname', 'John');
data.writeString('lastname', 'Doe');
headers.add('Content-Type', 'application/json');
response := http.post('http://api.examples.com', data);
```

## Http Client implementation

### Curl

To use curl based implementation, you need to add conditiona compilation define `{$DEFINE LIBCURL}` or via command line compiler argument `-dLIBCURL` and register any `THttp*Factory` class to dependency container.

```
var httpGet : IHttpGetClient;
...
container.add('http.get',THttpGetFactory.create());
...
httpGet := container['http.get'] as IHttpGetClient;
```

### TFPHttpClient

To use `TFPHttpClient` based implementation just register any `TFpcHttp*Factory` class to dependency container.

```
var httpGet : IHttpGetClient;
...
container.add('http.get',TFpcHttpGetFactory.create());
...
httpGet := container['http.get'] as IHttpGetClient;
```
### Null
Null class is provided that does nothing. It it provided so developer can disable http client.

To use null based implementation just register `TNullHttpClientFactory` class to dependency container.

```
var httpGet : IHttpGetClient;
...
container.add('http.get',TNullHttpClientFactory.create());
...
httpGet := container['http.get'] as IHttpGetClient;
```

## Explore more

- [Example web application that load data from Elasticsearch](https://github.com/fanoframework/fano-elasticsearch). This example use curl to call Elasticsearch API.
- [Example SCGI web application that load data from Elasticsearch](https://github.com/fanoframework/fano-elastic). This example use TFPHttpClient to call Elasticsearch API.
