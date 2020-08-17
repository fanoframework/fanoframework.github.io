---
title: Deploy as stand-alone web server
description: Tutorial on how to deploy Fano application as stand-alone web server.
---

<h1 class="major">Deployment as stand-alone Web Server</h1>

If you [create http web application](/scaffolding-with-fano-cli/creating-project#scaffolding-libmicrohttpd-project) such libmicrohttpd-based project using Fano CLI with `--project-mhd` command, you can simply access application directly from Internet browser using host and port where application listen, for example `http://localhost:8080`.

Other alternative is to run application behind reverse proxy web server such as Apache or Nginx.

## Deploy with Fano CLI

Simplest way to setup Fano web application with reverse proxy web server is to deploy http application using [Fano CLI](https://github.com/fanoframework/fano-cli), run with `--deploy-http=[domain name]`.

Inside Fano web application project directory, run

```
$ sudo fanocli --deploy-http=myapp.me
```

Command above, will create virtual host for Apache web server, enabled virtual host configuration, reload Apache web server configuration and add entry to `myapp.me` domain in `/etc/hosts`.

To setup for nginx web server add `--web-server=nginx`. Without it, it is assumed Apache web server.

```
$ sudo fanocli --deploy-http=myapp.me --web-server=nginx
```

If you want to setup manually without Fano CLI, read section below.

### Generate virtual host config to standard output

If you want to generate virtual host configuration without actually modifying
web server configuration, you can use `--stdout` command line option.
This option will generate virtual host configuration  and print it to standard output. It is useful if you want to deploy configuration manually.

Because it will not change any web server configuration, you do not need to run it with root privilege. So following code is suffice.

```
$ fanocli --deploy-http=myapp.me --stdout
```

## Apache with mod_proxy_http module

To deploy as http application with [mod_proxy_http](https://httpd.apache.org/docs/2.4/mod/mod_proxy_http.html), you need to have `mod_proxy_http` installed and loaded. This module is installed but not enabled by default.

### Debian

To enable module,

```
$ sudo a2enmod proxy_http
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
    ProxyPassMatch ^/(.*)$ http://127.0.0.1:20477
</VirtualHost>
```
You may need to replace `http://127.0.0.1:20477` with host and port where your
application is listening. If you use unix domain socket, you need to modify `ProxyPassMatch` as follows

```
ProxyPassMatch ^/(.*)$ "unix:/path/to/app.sock|http://127.0.0.1/"
```

Line `|http://127.0.0.1/` is required so `mod_proxy_http` is called to handle request, although, host and port information are ignored.


Two `ProxyPassMatch` lines tell Apache to serve requests for
files inside `css`, `images`, `js` directories directly. For other, pass requests to our application.

On Debian, save it to `/etc/apache2/sites-available` for example as `fano-http.conf`

Enable this site and reload Apache

```
$ sudo a2ensite fano-http.conf
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
        proxy_pass http://127.0.0.1:20477;
    }
}
```
Change `proxy_pass` to match host and port where application is listening.

Last two `location` configurations tells Nginx to serve files directly if exists, otherwise pass it to our application.

## Issue with firewall

In Fedora-based distribution, firewall is active by default. Read [Issue with firewall](/deployment/scgi#issue-with-firewall) for more information.

## Permission issue with SELinux

Running http application through reverse proxy may be subject to strict security policy of SELinux. Read [Permission issue with SELinux](/deployment/scgi#permission-issue-with-selinux) for more information.

## Explore more

- [Deploy as FastCGI application](/deployment/fastcgi)
- [Deploy as SCGI application](/deployment/scgi)
- [Deploy as uwsgi application](/deployment/uwsgi)
- [Deploy Fano application with load balancer](/deployment/load-balancer-setup)
- [Back to Deployment](/deployment)
