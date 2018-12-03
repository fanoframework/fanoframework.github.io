---
title: Working with Controllers
description: Tutorial on how to work with controllers in Fano Framework
---

<h1 class="major">Working with Controllers</h1>


## IRequestHandler interface

When web application receives request from web server, it will be given
request uri of resource that user is requesting.
Dispatcher parses request uri to find matching route handler that will handle this request. If no handler available to handle it, HTTP 404 will be returned.

Interface `IRequestHandler`, declared in unit `fano.pas`, is basis of route handler implementation in Fano Framework. It consists of `handleRequest()` method that implementor class must provide.
