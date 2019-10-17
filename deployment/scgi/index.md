---
title: Deploy as SCGI application
description: Tutorial on how to deploy SCGI web application built with Fano Framework to various web servers.
---

<h1 class="major">Deployment as SCGI application</h1>

Fano Framework can be deployed as [SCGI (Simple Common Gateway Interface)](https://python.ca/scgi/protocol.txt) web application. SCGI is similar to FastCGI protocol
but simpler to parse.

See [Working with Application](/working-with-application) for information how to create SCGI web application.

See [Fano SCGI](https://github.com/fanoframework/fano-scgi) for example application.
You may want to look [Scaffolding with Fano CLI](/scaffolding-with-fano-cli) to easily create new SCGI web application project.

## Deploy with Fano CLI

Simplest way to setup Fano web application with web server is to deploy SCGI application with [Fano CLI](https://github.com/fanoframework/fano-cli), run with `--deploy-scgi=[domain name]`.

Inside Fano web application project directory, run

```
$ sudo fanocli --deploy-scgi=myapp.me
```

Command above, will create virtual host for Apache web server, enabled virtual host configuration, reload Apache web server configuration and add entry to `myapp.me` domain in `/etc/hosts`.

To setup for nginx web server add `--web-server=nginx`. Without it, it is assumed Apache web server.

```
$ sudo fanocli --deploy-scgi=myapp.me --web-server=nginx
```

If you want to setup manually without Fano CLI, read section below.

## Apache with mod_proxy_scgi module

To deploy as SCGI application with [mod_proxy_scgi](https://httpd.apache.org/docs/2.4/mod/mod_proxy_scgi.html)

You need to have `mod_proxy_scgi` installed and loaded. This module is Apache's built-in module, so it is very likely that you will have it with your Apache installation. You just need to make sure it is loaded.

### Debian

For example, on Debian,

```
$ sudo a2enmod proxy_scgi
$ sudo systemctl restart apache2
```

Create virtual host config and add `ProxyPassMatch`, for example

```
<VirtualHost *:80>
     ServerName www.example.com
     DocumentRoot /home/example/public

     <Directory "/home/example/public">
         Options +ExecCGI
         AllowOverride FileInfo
         Require all granted
     </Directory>

    ProxyRequests Off
    ProxyPassMatch /(css|images|js).* !
    ProxyPassMatch ^/(.*)$ scgi://127.0.0.1:20477
</VirtualHost>
```
You may need to replace `scgi://127.0.0.1:20477` with host and port where your
application is running.

Two `ProxyPassMatch` lines tell Apache to serve requests for
files inside `css`, `images`, `js` directories directly. For other, pass requests to our application.

On Debian, save it to `/etc/apache2/sites-available` for example as `fano-scgi.conf`

Enable this site and reload Apache

```
$ sudo a2ensite fano-scgi.conf
$ sudo systemctl reload apache2
```

### Fedora

The difference between Debian and Fedora regarding Apache mostly about default service name (Debian uses `apache2` and Fedora uses `httpd`), user which Apache run (Debian uses `www-data` user while Fedora uses `apache`) and default Apache configuration location.

- Create virtual host

Apache main configuration is stored in `/etc/httpd/httpd.conf`. To add virtual host configuration, you can simply add entry to this file or prefered way is to create configuration in file `/etc/httpd/conf.d/` directory.

In Fedora and Apache 2.4, `/etc/httpd/conf.d/` directory will be search for additional configurations. So you can just create virtual host file inside this directory.

- Reload Apache service

Tell Apache to load configuration by running

```
$ sudo systemctl reload httpd
```

## Nginx

Create virtual host configuration file in `/etc/nginx/conf.d` directory, for example

```
server {
    listen 80;
    root /home/example.fano/public;
    server_name example.fano;
    error_log /var/log/nginx/example.fano-error.log;
    access_log /var/log/nginx/example.fano-access.log;

    location / {
        try_files $uri @example.fano;
    }

    location @example.fano {
        scgi_pass 127.0.0.1:20477;
        include scgi_params;
    }
}
```
Change `scgi_pass` to match host and port where application is listening.

Last two `location` configurations tells Nginx to serve files directly if exists, otherwise pass it to our application.


## <a name="issue-with-firewall">Issue with firewall

Fedora, Red Hat Enterprise Linux and Centos come with firewalld daemon enabled which block http and https port by default. To enable http/https traffic, first we need to get active zones. Run following command as root

```
# firewall-cmd --get-active-zones
```

In my Fedora, command above output something like below (yours may be different),

```
FedoraServer
  interfaces: enp0s3
```

To enable http and https, run

```
# firewall-cmd --zone=FedoraServer --permanent --add-service=http
# firewall-cmd --zone=FedoraServer --permanent --add-service=https
```

Reload firewalld by running,

```
# firewall-cmd --reload
```

## <a name="permission-issue-with-selinux">Permission issue with SELinux

Fedora, Red Hat Enterprise Linux and Centos come with SELinux enabled and with very strict security policy by default. This may pose permission issue when running SCGI application through reverse proxy as shown in following error log,

```
2019/10/17 15:17:58 [crit] 1022#0: *6 connect() to 127.0.0.1:20477 failed (13: Permission denied) while connecting to upstream, client: 192.168.0.79, server: example.fano, request: "GET / HTTP/1.1", upstream: "scgi://127.0.0.1:20477", host: "example.fano"
```

Simple solution was to run SELinux with `permissive` mode. In permissive mode, SELinux permits all operations but log operations that would have breached in `enforcing` mode.

Web server such Apache or Nginx is listed in SELinux under `httpd_t` context. Run following command as root to make add `httpd_t` to permissive mode.

```
# semanage permissive -a httpd_t
```

Nginx has detail information [how to fix issue with SELinux](https://www.nginx.com/blog/using-nginx-plus-with-selinux/). Read [Centos Wiki regarding SELinux](https://wiki.centos.org/HowTos/SELinux) for more information.

## Explore more

- [Deploy as FastCGI application](/deployment/fastcgi)
- [Deploy Fano application with Apache load balancer module](/deployment/load-balancer-setup)
- [Back to Deployment](/deployment)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
