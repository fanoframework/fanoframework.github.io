---
title: Deploy Fano application with Apache load balancer module
description: Tutorial on how to deploy Fano application with Apache mod_proxy_balancer.
---

<h1 class="major">Deploy Fano application with Apache load balancer module</h1>

Apache provides reverse proxy load balancer to distribute load to one or more application instances. When one application is unable to handle request, load balancer distributes load to other application instance thus improving performance, availability and scalability.

## Requirement

- [Apache 2.4](https://httpd.apache.org/docs/2.4/)
- [mod_proxy_balancer](https://httpd.apache.org/docs/2.4/mod/mod_proxy_balancer.html)
- [mod_proxy_scgi](https://httpd.apache.org/docs/2.4/mod/mod_proxy_scgi.html), [mod_proxy_fcgi](https://httpd.apache.org/docs/2.4/mod/mod_proxy_fcgi.html), [mod_proxy_uwsgi](https://httpd.apache.org/docs/2.4/mod/mod_proxy_uwsgi.html) or [mod_proxy_http](https://httpd.apache.org/docs/2.4/mod/mod_proxy_http.html)
- [Fano CLI](https://github.com/fanoframework/fano-cli)

## Setting up mod_proxy_balancer

Make sure that `mod_proxy_balancer` is enabled. For example, in Debian-based, run

```
$ sudo a2enmod proxy_balancer
```

Also you need to enable one or more [scheduler algorithm module](/deployment/load-balancer-setup#load-balancing-scheduler-algorithm). For example,

```
$ sudo a2enmod lbmethod_byrequests
```

## <a name="deploy-fano-application-with-load-balancer-with-fano-cli"></a>Deploy Fano Application with load balancer with Fano CLI

Apart from task for [scaffolding web application](/scaffolding-with-fano-cli),
[Fano CLI](https://github.com/fanoframework/fano-cli) also provides `--deploy-lb-scgi=[domain name]`, `--deploy-lb-fcgi=[domain name]`, `--deploy-lb-uwsgi=[domain name]` or `--deploy-lb-http=[domain name]` to help setup load balancer during development for SCGI, FastCGI, uwsgi or http web application, respectively.

After you create SCGI, FastCGI, uwsgi or http project with `--project-scgi`, `--project-fcgi`, `--project-uwsgi` or `--project-mhd`, for example

```
$ fanocli --project-scgi=myapp
```

From inside Fano SCGI web application project directory, run

```
$ sudo fanocli --deploy-lb-scgi=myapp.fano
```

Command above, will create virtual host for Apache web server that utilize `mod_proxy_balancer` module, enabled virtual host configuration, reload Apache web server configuration and add entry to `myapp.fano` domain in `/etc/hosts`.

Replace with `--deploy-lb-fcgi`, `--deploy-lb-uwsgi` or `--deploy-lb-http` for setting up FastCGI, uwsgi or http web application respectively.

## Deploy Fano Application with load balancer manually

Skip this section if you deploy using Fano CLI.

If you prefer setting up virtual host manually, create new file in `/etc/httpd/conf.d` or `/etc/apache/sites-available` directory for Fedora-based or Debian-based Linux respectively.

Add, for example, following code,

```
<VirtualHost *:80>
    ServerName myapp.fano
    DocumentRoot /path/to/my/app/doc/root

    ErrorLog /var/nginx/log/myapp.fano-error.log
    CustomLog /var/nginx/log/myapp.fano-access.log combined

    <Directory /path/to/my/app/doc/root>
        Options -MultiViews -FollowSymlinks +SymlinksIfOwnerMatch +ExecCGI
        AllowOverride FileInfo Indexes
        Require all granted
    </Directory>

    <Proxy balancer://myapp.fano>
        BalancerMember scgi://127.0.0.1:20477
        BalancerMember scgi://127.0.0.1:20478
        ProxySet lbmethod=by_requests
    </Proxy>

    ProxyRequests Off
    ProxyPassMatch "/css|js|images|img|plugins|bower_components(.*)" !
    ProxyPassMatch ^/(.*)$ "balancer://myapp.fano"
</VirtualHost>
```

Replace `BalancerMember` url according to protocol, host and port of the application. For example, replace with `fcgi://127.0.0.1:20477`, `uwsgi://127.0.0.1:20477` or `http://127.0.0.1:20477` for FastCGI, uwsgi or http protocol.

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

Open `http://myapp.fano` you should see main controller is invoked. There is no visual indication compare to application running without load balancer.

Indication that your application is running with `mod_proxy_balancer` is availability of new environment variables, for examples `BALANCER_NAME`, `BALANCER_WORKER_NAME` and `BALANCER_WORKER_ROUTE`.

Try to access route that does not exist, so that environment variables are printed on the browser. It will show that load balancer distribute the load by observing value of `BALANCER_WORKER_NAME` variable.

## Set balancer member

By default, if parameter `--members` not set, it is assumed that you will use two application instances, running on `127.0.0.1:20477` and `127.0.0.1:20478` respectively.

`--members` parameter allows set multiple balancer members separated by coma.

```
$ sudo fanocli --deploy-lb-scgi=myapp.fano --members=127.0.0.1:20000,localhost:20001
```

Run applications at once with consecutive listening ports.

```
$ ./bin/app.cgi --host=127.0.0.1 --port=20000 & ./bin/app.cgi --host=localhost --port=20001 &
```

Developer is responsible to make sure that no other application is using those listening ports.

## <a name="load-balancing-scheduler-algorithm"></a>Change load balancing scheduler algorithm

By default, if parameter `--lbmethod` is not set, `byrequests` is assumed, which will use `mod_lbmethod_byrequests` module to distribute requests. Please refer to [mod_proxy_balancer documentation](https://httpd.apache.org/docs/2.4/mod/mod_proxy_balancer.html) for available algorithms.

For example, to use `mod_lbmethod_bybusyness` module,

```
$ sudo fanocli --deploy-lb-scgi=myapp.fano --lbmethod=bybusyness
```

## Explore more

- [Deploy Fano application with Nginx load balancer module](/deployment/load-balancer-setup/nginx)
- [Hello World SCGI application with Fano CLI](/tutorials/hello-world-scgi-application-with-fano-cli)
- [Deploy as FastCGI application](/deployment/fastcgi)
- [Deploy as SCGI application](/deployment/scgi)
- [Deploy as uwsgi application](/deployment/uwsgi)
- [Deploy as standalone web server](/deployment/standalone-web-server)
- [Back to Deployment](/deployment)
