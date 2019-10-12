---
title: Deployment
description: Tutorial on how to deploy web application built with Fano Framework to various web servers.
---

<h1 class="major">Deployment</h1>

## Executable binary file permission

You need to make sure that application executable binary has proper file permission. It needs execution bit to be set.

```
$ chmod 744 /path/to/app.cgi
```
For most uses-cases, `744` permission is suffice if your web server is run as your user account. If web server run as separate user than owner of cgi file, for example as `www-data`. You can add `www-data` as group

```
$ sudo chown my_user:www-data /path/to/app.cgi
```

and then set executable binary permission to `774`.

If you do not have root privilege access, such in shared hosting environment, then use permission of `755`. It will make application binary executable to everyone.

## Deploy to various setup

- [Deploy as CGI application](/deployment/cgi)
- [Deploy as FastCGI application](/deployment/fastcgi)
- [Deploy as SCGI application](/deployment/scgi)
- [Deploy as standalone web server](/deployment/standalone-web-server)
- [Deploy using Docker](/deployment/docker)
- [Deploy with Continous Integration tools](/deployment/continous-integration)
- [Deploy Fano application with Apache load balancer module](/deployment/load-balancer-setup)
