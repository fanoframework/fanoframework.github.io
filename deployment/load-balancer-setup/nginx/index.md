---
title: Deploy Fano web application with Nginx load balancer module
description: Tutorial on how to deploy Fano web application with Nginx load balancer module.
---

<h1 class="major">Deploy Fano web application with Nginx load balancer module</h1>

Nginx provides reverse proxy load balancer to distribute load to one or more application instances. When one application is unable to handle request, load balancer distributes load to other application instance thus improving performance, availability and scalability.

## Requirement

- [Nginx](https://nginx.org/)
- [Fano CLI](https://github.com/fanoframework/fano-cli)

## Deploy Fano Application with load balancer with Fano CLI

Apart from task for [scaffolding web application](/scaffolding-with-fano-cli),
[Fano CLI](https://github.com/fanoframework/fano-cli) also provides `--deploy-lb-scgi=[domain name]` and `--deploy-lb-fcgi=[domain name]` to help setup load balancer during development for SCGI and FastCGI web application, respectively.

After you create SCGI or FastCGI project with `--project-scgi` or `--project-fcgi`,

```
$ fanocli --project-scgi=myapp
```

From inside Fano SCGI web application project directory, run

```
$ sudo fanocli --deploy-lb-scgi=myapp.fano --web-server=nginx
```

Command above, will create virtual host for Nginx web server that utilize Nginx load balancer module, reload Nginx web server configuration and add entry to `myapp.fano` domain in `/etc/hosts`.

Replace with `--deploy-lb-fcgi` for setting up FastCGI web application with load balancer.

## Running multiple applications with load balancer

Build the application and then run two applications at once with consecutive listening ports.

```
$ ./build.sh
$ ./bin/app.cgi --port=20477 & ./bin/app.cgi --port=20478 &
```

`&` is required to make sure that both applications is running in background.

## Access application from browser

Open `http://myapp.fano` you should see main controller is invoked.

## Set balancer member

By default, if parameter `--members` not set, it is assumed that you will use two application instances, running on `127.0.0.1:20477` and `127.0.0.1:20478` respectively.

`--members` parameter allows set multiple balancer members separated by coma.

```
$ sudo fanocli --deploy-lb-scgi=myapp.fano --web-server=nginx --members=127.0.0.1:20000,localhost:20001
```

Run applications at once with consecutive listening ports.

```
$ ./bin/app.cgi --host=127.0.0.1 --port=20000 & ./bin/app.cgi --host=localhost --port=20001 &
```

Developer is responsible to make sure that no other application is using those listening ports.

## <a name="load-balancing-scheduler-algorithm"></a>Change load balancing scheduler algorithm

By default, if parameter `--lbmethod` is not set, round-robin algorithm is assumed, which will distribute requests in round-robin fashion. Please refer to [Nginx load balancer documentation](http://nginx.org/en/docs/http/load_balancing.html) for available algorithms.

For example, to use least connected load balancing, set to `least_conn`,

```
$ sudo fanocli --deploy-lb-scgi=myapp.fano --lbmethod=least_conn
```

## Explore more

- [Hello World SCGI application with Fano CLI](/tutorials/hello-world-scgi-application-with-fano-cli)
- [Deploy as FastCGI application](/deployment/fastcgi)
- [Deploy as SCGI application](/deployment/scgi)
- [Back to Deployment](/deployment)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
