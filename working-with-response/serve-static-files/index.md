---
title: Serving static files
description: Tutorial on how to serve static files
---

<h1 class="major">Serving static files</h1>

Web application needs to serve static files that seldom change during
request response cycle such as JavaScript codes, cascading stylesheet files and also fonts and images.

## Serve static files with reverse proxy server
If your application is running behind reverse proxy server, easiest way to serve static files is to let reverse proxy server does it for you. Thus your appplication can focus on problem your application is trying to solve.

For example if you use nginx, you can use nginx `try` which will try to serve files if it exists. If it does not then let your application handles it.

If your application is deployed using Fano CLI, it adds web server configuration that tells web server to serves static files on behalf of application.

## Serve static files within application

There are times when you need to serve files on your own, for example, you need to check if current user is authorized to download file.

To serve static file, you can return instance of `TFileResponse`.