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

This is mostly similar to [deploy Fano web application with Apache load balancer module with Fano CLI](/deployment/load-balancer-setup/apache#deploy-fano-application-with-load-balancer-with-fano-cli).

The difference is that you need to pass `--web-server=nginx` parameter when deploying, for example

```
$ sudo fanocli --deploy-lb-scgi=myapp.fano --web-server=nginx
```

Replace with `--deploy-lb-fcgi`, `--deploy-lb-uwsgi` or `--deploy-lb-http` for setting up FastCGI, uwsgi or http web application respectively.

### Generate virtual host config to standard output

If you want to generate virtual host configuration without actually modifying
web server configuration, you can use `--stdout` command line option.
This option will generate virtual host configuration  and print it to standard output. It is useful if you want to deploy configuration manually.

Because it will not change any web server configuration, you do not need to run it with root privilege. So following code is suffice.

```
$ fanocli --deploy-lb-fcgi=myapp.fano --web-server=nginx --stdout
```

## Deploy Fano Application with load balancer manually

Skip this section if you use Fano CLI to deploy application.

If you prefer setting up virtual host manually, create new file in `/etc/nginx/conf.d` or `/usr/local/etc/nginx/conf.d` directory for Linux or FreeBSD respectively and add, for example, following code,

```
upstream my-app-load-balancer {
    server 127.0.0.1:20477;
    server 127.0.0.1:20478;
}

server {
    listen 80;
    root /path/to/my/app/doc/root;
    server_name myapp.fano;

    error_log /var/nginx/log/myapp.fano-error.log;
    access_log /var/nginx/log/myapp.fano-access.log;

    location / {
        try_files $uri @myapp.fano;
    }

    location @myapp.fano {
        scgi_pass my-app-load-balancer;
        include scgi_params;
    }
}
```

Replace `scgi_pass` and `scgi_params` with `fastcgi_pass` and `fastcgi_params` for FastCGI or
`uwsgi_pass` and `uwsgi_params` for uwsgi web application.

For http web application, configuration is similar except that you need to replace `location` with

```
    location @myapp.fano {
        proxy_pass http://my-app-load-balancer;
    }
```

Reload Nginx

```
$ sudo systemctl reload nginx
```

Add entry to `/etc/hosts` file so you can access application during development,

```
127.0.0.1 myapp.fano
```

## Running multiple applications with load balancer

Build the application and then run two applications at once with consecutive listening ports.

```
$ ./build.sh
$ ./bin/app.cgi --port=20477 & ./bin/app.cgi --port=20478 &
```

`&` is required to make sure that both applications are running in parallel.

To stop all applications,

```
$ pkill app.cgi
```
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

- [Deploy Fano application with Apache load balancer module](/deployment/load-balancer-setup/apache)
- [Hello World SCGI application with Fano CLI](/tutorials/hello-world-scgi-application-with-fano-cli)
- [Deploy as FastCGI application](/deployment/fastcgi)
- [Deploy as SCGI application](/deployment/scgi)
- [Back to Deployment](/deployment)
