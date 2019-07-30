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
var params : IList;
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
var cookies : IList;
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
var params : IList;
...
params := request.getParsedBodyParams();
```

## Handling file upload

If you have form like following snippet

```
<form action="/submit" method="post" enctype="multipart/form-data">
    <input type="text" name="username" value="test">
    <input type="file" name="myFile">
    <button type="submit">Submit</button>
</form>
```

To retrieve value of `username` is same as with POST data.

```
var username : string;
...
username := request.getParsedBodyParam('username');
```

To retrieve file uploaded by client, you need to call `getUploadedFile()`. It returns data of type `IUploadedFileArray` which array of `IUploadedFile` instance.


```
var myFile : IUploadedFileArray;
...
myFile := request.getUploadedFile('myFile');
```

Client can send one or more file with same name, for example:

```
<form action="/submit" method="post" enctype="multipart/form-data">
    <input type="text" name="username" value="test">
    <input type="file" name="myFile">
    <input type="file" name="myFile">
    <button type="submit">Submit</button>
</form>
```

If client upload one or more files, `length(myFile)` will indicates how many
files actually uploaded.

### Move uploaded file

Initially, uploaded file will be stored in system temporary directory. You need to
call `moveTo()` methods to move into permanent location.

```
if (length(myFile) > 0) then
begin
    myFile[0].moveTo(targetUploadPath);
end;
```
`targetUploadPath` is full filename (including its target directory). You must make sure that `targetUploadPath` is writeable. Exception `EInOutError` is raised when
uploaded file can not be written to target path.

### Get uploaded file size

Size of uploaded file in octet (or bytes) can be queried with `size()` method.

```
var fSize : int64;
...
fSize := myFile[0].size();
```

### Get file MIME type

MIME type of uploaded file can be queried with `getClientMediaType()` method.
For example, if upload JPEG image, you may get `image/jpeg`.

```
var mime : string;
...
mime := myFile[0].getClientMediaType();
```

If client does not send any MIME type (aka Content Type) then it is assumed
`application/octet-stream`.

### Get original file name

To get original file name of uploaded file use `getClientFilename()` method.

```
var filename : string;
...
filename := myFile[0].getClientFilename();
```

If client does not send filename then it returns empty string.

For example how to handle file upload with Fano Framework, see
[Fano Upload](https://github.com/fanoframework/fano-upload).

## Explore more

- [Working with response](/working-with-response)
- [Working with Controllers](/working-with-controllers)
- [Working with Views](/working-with-views)
