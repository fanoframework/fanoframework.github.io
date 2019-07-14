---
title: Deploy as CGI application
description: Tutorial on how to deploy CGI web application built with Fano Framework to various web servers.
---

<h1 class="major">Deployment as CGI application</h1>

### Apache

This section explains how to deploy web application as CGI application on Apache web server.

#### Dedicated server

This section explains how to deploy server that you have full control and administrative privilege (aka root access).

##### Debian-based Linux

- Create virtual host by adding Apache configuration file inside `/etc/apache2/sites-available` and put

```
<VirtualHost *:80>
    ServerAdmin [[server administrator email]]
    ServerName [[fano application hostname]]

    DocumentRoot "[[/path/to/fano/app/public]]"

    <Directory "[[/path/to/fano/app/public]]">
        Options -MultiViews +ExecCGI
        AllowOverride FileInfo Indexes
        Require all granted
        AddHandler cgi-script .cgi
        DirectoryIndex app.cgi
     </Directory>
 </VirtualHost>
```

Replace `[[server administrator email]]` with administrator email for example, if we use domain name `fano.dev` we can use `admin@fano.dev`.

Replace `[[fano application hostname]]` with domain that we use,
for example in this case, it is `fano.dev`. So when Apache receive request with URI http://fano.dev/test it will match
this virtual host entry and call our application.


Replace `[[/path/to/fano/app/public]]` with directory where our application binary reside. If you store it outside Apache
document root directory, you need to specify with with `<Directory>`.

For example, if we store application binary in directory `/home/myuser/myapp/public` which outside Apache document root, we need to
add `Require` directive to allow Apache to serve request from
this directory, otherwise they will be rejected.

Above snippet basically tells Apache to allow (`+ExecCGI`) to serve any file ends with `cgi` extension as CGI script (`AddHandler cgi-script .cgi`). `-MultiViews` also important to disable automatic content negotiation because we need to add URL rewriting to allow use of routing in our application. So Our application will be responsible to decide what response to return.

So in our example, we end up with following configuration:

```
<VirtualHost *:80>
    ServerAdmin admin@fano.dev
    ServerName fano.dev

    DocumentRoot "/home/myuser/myapp/public"

    <Directory "/home/myuser/myapp/public">
        Options -MultiViews +ExecCGI
        AllowOverride FileInfo Indexes
        Require all granted
        AddHandler cgi-script .cgi
        DirectoryIndex app.cgi
     </Directory>
 </VirtualHost>
```

You need to make sure that `app.cgi` has execution bit, if not then run

```
$ chmod 744 /home/myuser/myapp/public/app.cgi
```
For most uses-cases, `744` permission is suffice if your web server is run as your user account. If executable binary failed to run, try `774`

Save it as `fano.conf` file. Note that saving data to `/etc/apache2/sites-available` directory requires administrative privilege.

- Enable virtual host

To make our virtual host active, put symlink in `/etc/apache2/sites-enabled` that points to `/etc/apache2/sites-available/fano.conf`.

```
$ sudo ln -s /etc/apache2/sites-available/fano.conf /etc/apache2/sites-enabled/fano.conf
```
or you can use `a2ensite` utility that comes with Debian distribution which does same thing.

```
$ sudo a2ensite fano.conf
```

- Restart Apache service

```
$ sudo service apache2 restart
```

If configuraton is ok, Apache will restart happily.

- Setup domain name to DNS

If you have not register a domain name with domain name registrar, you can setup fake domain name for local development only by adding
entry to `/etc/hosts` file. Open `/etc/hosts` and add following entry,

```
127.0.0.1 fano.dev
```

`127.0.0.1` with assumption that Apache run on same machine.

##### Fedora-based Linux

#### Shared-hosting

This section explains how to deploy server that you have no full control and have very limited administrative privelege.

##### Debian-based Linux

##### Fedora-based Linux

##### CPanel-based server

### Nginx

This section explains how to deploy web application as CGI application on Nginx web server.
