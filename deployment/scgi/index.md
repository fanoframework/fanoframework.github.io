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
    ProxyPass /css !
    ProxyPass /images !
    ProxyPass /js !
    ProxyPassMatch ^/(.*)$ scgi://127.0.0.1:20477
</VirtualHost>
```
You may need to replace `scgi://127.0.0.1:20477` with host and port where your
application is running.

Last four line of virtual host configurations basically tell Apache to serve any
files inside `css`, `images`, `js` directly otherwise pass it to our application.

On Debian, save it to `/etc/apache2/sites-available` for example as `fano-scgi.conf`

Enable this site and restart Apache

```
$ sudo a2ensite fano-scgi.conf
$ sudo systemctl restart apache2
```


### Fedora



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
        scgi_pass 127.0.0.1:20477;
        include scgi_params;

    }

    location ~ /\.ht {
        deny all;
    }
}
```
Change `scgi_pass` to match host and port where application is listening.

[Back to Deployment](/deployment)
