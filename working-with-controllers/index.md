---
title: Working with Controllers
description: Tutorial on how to work with controllers in Fano Framework
---

<h1 class="major">Working with Controllers</h1>

## IRequestHandler and IRouteHandler interface

When web application receives request from web server, it will be given
request uri of resource that user is requesting.
Dispatcher parses request uri to find matching route handler that will handle this request. If no handler available to handle it, HTTP 404 will be returned.

Interface `IRequestHandler`, declared in unit `fano.pas`, is basis of route handler implementation in Fano Framework. It consists of `handleRequest()` method that implementor class must provide.

Interface `IRouteHandler`, is extension to `IRequestHandler` interface and provide more functionality such get route argument data or set middleware for route.

`IRouteHandler` is basis of all route handler. When you set route, you must pass instance of class that implements `IRouteHandler`.

Fano Framework provides base controller in `TController` class that implements `IRouteHandler`. But of course, you are free to implements your own.

## Using TController class

`TController` class is built-in class that provides ability for route handler to
works with view and middlewares.

Except for simple route handler which display static view, you are very likely need to extends this class as `TController` by defaut, does very little job, which is
returning response from `IView` instance.