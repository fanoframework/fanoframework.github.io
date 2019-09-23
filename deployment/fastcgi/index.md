---
title: Deploy as FastCGI application
description: Tutorial on how to deploy FastCGI web application built with Fano Framework to various web servers.
---

<h1 class="major">Deployment as FastCGI application</h1>

Fano Framework can be deployed as [FastCGI](https://fastcgi-archives.github.io/FastCGI_Specification.html) web application.

See [Working with Application](/working-with-application) for information how to create FastCGI web application.

See example application:

- [Fano FastCGI](https://github.com/fanoframework/fano-fastcgi), example FastCGI application which listening on TCP host:port
- [Fano FCGI Unix](https://github.com/fanoframework/fano-fcgi-unix), example FastCGI application which listening using Unix Domain Socket.
- [Fano fcgid](https://github.com/fanoframework/fano-fcgid), example FastCGI application which started automatically by [Apache mod_fcgid](https://httpd.apache.org/mod_fcgid/mod/mod_fcgid.html) module.

You may want to look [Scaffolding with Fano CLI](/scaffolding-with-fano-cli) to easily
create new FastCGI web application project.

## Deploy with Fano CLI

Simplest way to setup Fano web application with web server is to deploy FastCGI application with [Fano CLI](https://github.com/fanoframework/fano-cli), run with `--deploy-fcgi=[domain name]` or `--deploy-fcgid=[domain name]`. The first command will setup FastCGI virtual host that works with `mod_proxy_fcgi` while the other for `mod_fcgid`.

Inside Fano web application project directory, run

```
$ sudo fanocli --deploy-fcgi=myapp.me
```

Command above, will create virtual host for Apache web server, enabled virtual host configuration, reload Apache web server configuration and add entry to `myapp.me` domain in `/etc/hosts`.

To setup for nginx web server add `--web-server=nginx`. Without it, it is assumed Apache web server.

```
$ sudo fanocli --deploy-fcgi=myapp.me --web-server=nginx
```

To deploy FastCGI web application for Apache mod_fcgid, run with `--deploy-fcgid`

```
$ sudo fanocli --deploy-fcgid=myapp.me
```

## Apache with mod_proxy_fcgi module

To deploy as FastCGI application with [mod_proxy_fcgi](https://httpd.apache.org/docs/2.4/mod/mod_proxy_fcgi.html)

You need to have `mod_proxy_fcgi` installed and loaded. This module is Apache's built-in module, so it is very likely that you will have it with your Apache installation. You just need to make sure it is loaded.

### Debian

For example, on Debian,

```
$ sudo a2enmod proxy_fcgi
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
    ProxyPassMatch ^/(.*)$ fcgi://127.0.0.1:20477
</VirtualHost>
```
You may need to replace `fcgi://127.0.0.1:20477` with host and port where your
application is running.

Two `ProxyPassMatch` lines tell Apache to serve requests for
files inside `css`, `images`, `js` directories directly. For other, pass requests to our application.

On Debian, save it to `/etc/apache2/sites-available` for example as `fano-fastcgi.conf`
Enable this site and restart Apache

```
$ sudo a2ensite fano-fastcgi.conf
$ sudo systemctl restart apache2
```

#### Use unix domain socket

If your Fano application and Apache run on same machine, you can get small improvement by using unix domain socket file. See example [Fano Fastcgi Unix application](https://github.com/fanoframework/fano-fcgi-unix).

If application is listening using socket file `/tmp/fano-fcgi.sock`, you need to change `ProxyPassMatch` as follows:

```
ProxyPassMatch ^/(.*)$ "unix:/tmp/fano-fcgi.sock|fcgi://127.0.0.1/"
```
Note that `|fcgi://127.0.0.1/` is required so `mod_proxy_fcgi` is called to handle request, although, host and port information are ignored.

#### Socket file permission issue

You need to make sure that `/tmp/fano-fcgi.sock` is writeable by Apache.

If you run with something like

```
$ ./bin/app.cgi
```
`/tmp/fano-fcgi.sock` file will be own by user where above command is executed. It will cause permission denied for Apache user.

For development stage, easy solution is just change owner of file. After application is bound and listen, you may want to open another
shell and execute

```
$ sudo chown [your current user]:www-data /tmp-fano-fcgi.sock
```
where `www-data` is user which Apache runs.

Using `/tmp` also may cause problem if you run Debian 9 - based distribution (such as Ubuntu 18.04). In Debian 9 (Stretch) or newer, each user, by default, has private /tmp directory. So if you run application as current user, `/tmp/fano-fcgi.sock` will not be found by Apache which run as different user.

Proper way is setup application to run as service, set socket file to `/var/run` and set group which service run, add Apache user into that group. Then you can
just run application using

```
$ sudo systemctl start [your-app-service-name]
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

## Apache with mod_fcgid module

To deploy as FastCGI application with [mod_fcgid](https://httpd.apache.org/mod_fcgid/mod/mod_fcgid.html), make sure you use `TSimpleSockFastCGIWebApplication` as base application.
Internally, it uses `TBoundSocketSvrImpl` as socket server.

Unlike `mod_proxy_fcgi` module where our application is run independently,
`mod_fcgid` provides automatic process management. So our application process lifecycle is managed by this module. It will spawn or kill one or more our application processes based on request load.

Socket connection is already bound and listen by `mod_fcgid`. so we need to
use `TBoundSocketSvrImpl` as socket server.

and of course you need to have `mod_fcgid` installed and loaded.

### Debian

To install `mod_fcgid`,

```
$ sudo apt-get install libapache2-mod-fcgid
```

To enable it,

```
$ sudo a2enmod fcgid
$ sudo systemctl restart apache2
```

Create virtual host config and add `fcgid-script`, for example

```
<VirtualHost *:80>
     ServerName www.example.com
     DocumentRoot /home/example/public

     <Directory "/home/example/public">
        Options +ExecCGI
        AllowOverride FileInfo
        Require all granted
        AddHandler fcgid-script .cgi
        DirectoryIndex app.cgi
     </Directory>
</VirtualHost>
```

Configuration above basically tells Apache to give any request to *.cgi file to
be handled by `mod_fcgid` module (identified by `fcgid-script` handler).

## Nginx

### Debian

Create virtual host configuration in `/etc/nginx/sites-available`, for example

```
server {
    listen 80;
    root /home/example/public;
    index index.html;
    server_name example.com;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.*$ {
        fastcgi_pass 127.0.0.1:20477;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```
Change `fastcgi_pass` to match host and port where application is listening.

If listening using unix domain socket, you need to change `fastcgi_pass` to

```
fastcgi_pass unix:/tmp/fano-fcgi.sock;
```

where `/tmp/fano-fcgi.sock` is socket file which application using and of course it must be writeable by nginx.

[Back to Deployment](/deployment)
