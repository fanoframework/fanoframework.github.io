---
title: Deploy as uwsgi application
description: Tutorial on how to deploy uwsgi web application built with Fano Framework to various web servers.
---

<h1 class="major">Deployment as uwsgi application</h1>

Fano Framework can be deployed as [uwsgi](https://uwsgi-docs.readthedocs.io/en/latest/Protocol.html) web application. uwsgi is binary protocol similar to FastCGI protocol
but simpler to parse.

See *[Working with Application](/working-with-application)* for information how to create uwsgi web application.

See [Fano uwsgi](https://github.com/fanoframework/fano-uwsgi) for example application.
You may want to look *[Scaffolding with Fano CLI](/scaffolding-with-fano-cli)* to easily create new uwsgi web application project.

## Deploy with Fano CLI

Simplest way to setup Fano web application with web server is to deploy uwsgi application with [Fano CLI](https://github.com/fanoframework/fano-cli), run with `--deploy-uwsgi=[domain name]`.

Inside Fano web application project directory, run

```
$ sudo fanocli --deploy-uwsgi=myapp.me
```

Command above, will create virtual host for Apache web server, enabled virtual host configuration, reload Apache web server configuration and add entry to `myapp.me` domain in `/etc/hosts`.

To setup for nginx web server add `--web-server=nginx`. Without it, it is assumed Apache web server.

```
$ sudo fanocli --deploy-uwsgi=myapp.me --web-server=nginx
```

If you want to setup manually without Fano CLI, read section below.

### Skip adding domain name entry in /etc/hosts

By default `--deploy-*` parameter will cause domain name entry is added in `/etc/hosts` file. You may want to setup domain name with DNS server manually or you do not want to mess up with `/etc/hosts` file. You can avoid it by adding `--skip-etc-hosts` parameter.

```
$ sudo fanocli --deploy-uwsgi=myapp.me --skip-etc-hosts
```

### Generate virtual host config to standard output

If you want to generate virtual host configuration without actually modifying
web server configuration, you can use `--stdout` command line option.
This option will generate virtual host configuration  and print it to standard output. It is useful if you want to deploy configuration manually.

Because it will not change any web server configuration, you do not need to run it with root privilege. So following code is suffice.

```
$ fanocli --deploy-uwsgi=myapp.me --stdout
```
## Change host and port

By default, Fano CLI, `--deploy-uwsgi` parameter will use `127.0.0.1` and `20477` as default host and port respectively. To use different value, you can edit generated virtual host configuration file or use `--host`, `--port` parameters when using `--deploy-uwsgi`.

```
$ sudo fanocli --deploy-uwsgi=myapp.me --host=192.168.2.2 --port=4000
```

## Apache with mod_proxy_uwsgi module

To deploy as uwsgi application with [mod_proxy_uwsgi](https://httpd.apache.org/docs/2.4/mod/mod_proxy_uwsgi.html), you need to have `mod_proxy_uwsgi` installed and loaded. This module is not installed by default.

### Debian

To install module on Debian,

```
$ sudo apt install libapache2-mod-proxy-uwsgi
```

To enable module,

```
$ sudo a2enmod proxy_uwsgi
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
    ProxyPassMatch ^/(.*)$ uwsgi://127.0.0.1:20477
</VirtualHost>
```
You may need to replace `uwsgi://127.0.0.1:20477` with host and port where your
application is running. If you use unix domain socket, you need to modify `ProxyPassMatch` as follows

```
ProxyPassMatch ^/(.*)$ "unix:/path/to/app.sock|uwsgi://127.0.0.1/"
```

Line `|uwsgi://127.0.0.1/` is required so `mod_proxy_uwsgi` is called to handle request, although, host and port information are ignored.

Two `ProxyPassMatch` lines tell Apache to serve requests for
files inside `css`, `images`, `js` directories directly. For other, pass requests to our application.

On Debian, save it to `/etc/apache2/sites-available` for example as `fano-uwsgi.conf`

Enable this site and reload Apache

```
$ sudo a2ensite fano-uwsgi.conf
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
        uwsgi_pass 127.0.0.1:20477;
        include uwsgi_params;
    }
}
```
Change `uwsgi_pass` to match host and port where application is listening.

Last two `location` configurations tells Nginx to serve files directly if exists, otherwise pass it to our application.

## Issue with firewall

In Fedora-based distribution, firewall is active by default. Read *[Issue with firewall](/deployment/scgi#issue-with-firewall)* for more information.

## Permission issue with SELinux

Running uwsgi application through reverse proxy may be subject to strict security policy of SELinux. Read *[Permission issue with SELinux](/deployment/scgi#permission-issue-with-selinux)* for more information.

## Explore more

- [Deploy as FastCGI application](/deployment/fastcgi)
- [Deploy as SCGI application](/deployment/scgi)
- [Deploy as standalone web server](/deployment/standalone-web-server)
- [Deploy Fano application with Apache load balancer module](/deployment/load-balancer-setup)
- [Back to Deployment](/deployment)
