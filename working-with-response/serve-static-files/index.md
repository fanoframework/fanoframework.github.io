---
title: Serving static files
description: Tutorial on how to serve static files
---

<h1 class="major">Serving static files</h1>

Web application needs to serve static files that seldom change during
request response cycle such as JavaScript codes, cascading stylesheet files and also fonts and images.

## Serve static files with reverse proxy server
If your application is running behind reverse proxy server, easiest way to serve static files is to let reverse proxy server does it for you. Thus your appplication can focus on problem it supposes to solve.

For example if you use Nginx, you can use `try_files` which will try to serve files if it exists. If it does not then let your application handles it.

```
server {
    ...
    location / {
        try_files $uri @myapp;
    }

    location @myapp {
        fcgi_pass 127.0.0.1:9000
        include fcgi_params;
    }

}
```

If your application is deployed using [Fano CLI](/deployment), it adds web server configuration which tells web server to serves static files on behalf of application.

## Serve static files within application

There are times when you need to serve files on your own, for example, you need to check if user is authorized to download file.

To serve static file in [controller](/working-with-controllers), you can return instance of `TFileResponse` as shown in following example.

```
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
var filename : string;
begin
    filename := fBaseDirectory + request.uri().getPath();
    if fileExists(filename) then
    begin
        //serve file
        result := TFileResponse.create(
            response.headers(),
            //ext .pdf returns content type application/pdf
            //ext .png returns content type image/png
            //and so on
            getContentTypeFromFilename(filename),
            filename
        );
    end else
    begin
        raise ENotFound.create(filename + ' not found');
    end;
end;
```

`TStaticFilesMiddleware` is [middleware](/middlewares) implementation which can serve static files if they exists. You can [attach it application middleware](/middlewares#attaching-middleware-to-application-middleware) list so that it applies to all [routes](/working-with-router).

```
var mimeTypes : IKeyValuePair;
...
mimeTypes := TKeyValuePair.create();

//register all file types you need to handle
mimeTypes.setValue('pdf', 'application/pdf');
mimeTypes.setValue('jpg', 'image/jpg');
mimeTypes.setValue('png', 'image/png');

appMiddlewares.add(
    TStaticFilesMiddlewares.create(
        '/path/to/static/files/base/directory',
        mimeTypes
    )
);
```

or registering it in [dependency container](/dependency-container) using its factory class `TStaticFilesMiddlewareFactory`,

```
container.add(
    'staticFiles',
    TStaticFilesMiddlewareFactory.create()
        .baseDirectory('/path/to/static/files/base/directory')
        .addMimeType('pdf', 'application/pdf')
        .addMimeType('jpg', 'image/jpg')
        .addMimeType('png', 'image/png')
);
...
appMiddlewares.add(container['staticFiles'] as IMiddleware);
```

`TStaticFilesMiddleware` decides to serve file based on existence of file. If you need more elaborate checks, you can inherit this middleware and override its `handleRequest()` method.

You can use several middlewares in case of multiple base directories. For example
you serve PDF files from different path than image files.

```
container.add(
    'imageStaticFilesMw',
    TStaticFilesMiddlewareFactory.create()
        .baseDirectory('/path/to/image/files/base/directory')
        .addMimeType('jpg', 'image/jpg')
        .addMimeType('png', 'image/png')
);
container.add(
    'pdfStaticFilesMw',
    TStaticFilesMiddlewareFactory.create()
        .baseDirectory('/path/to/pdf/files/base/directory')
        .addMimeType('pdf', 'application/pdf')
);
...
appMiddlewares
    .add(container['imagesStaticFilesMw'] as IMiddleware);
    .add(container['pdfStaticFilesMw'] as IMiddleware);
```
## Explore more

- [Middleware example application](/examples)
- [Working with Router](/working-with-router)
- [Middlewares](/middlewares)
