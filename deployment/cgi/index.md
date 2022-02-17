---
title: Deploy as CGI application
description: Tutorial on how to deploy CGI web application built with Fano Framework to various web servers.
---

<h1 class="major">Deployment as CGI application</h1>

## Deploy with Fano CLI

Simplest way to setup Fano web application with web server is to deploy CGI application with [Fano CLI](https://github.com/fanoframework/fano-cli), run with `--deploy-cgi=[domain name]`.

Inside Fano web application project directory, run

```
$ sudo fanocli --deploy-cgi=myapp.me
```

Command above, will create virtual host for Apache web server, enabled virtual host configuration, reload Apache web server configuration and add entry to `myapp.me` domain in `/etc/hosts`.

Because nginx does not support running CGI program, when you use `--web-server=nginx`, it does nothing.

### Skip adding domain name entry in /etc/hosts

By default `--deploy-cgi` parameter will cause domain name entry is added in `/etc/hosts` file. You may want to setup domain name with DNS server manually or you do not want to mess up with `/etc/hosts` file. You can avoid it by adding `--skip-etc-hosts` parameter.

```
$ sudo fanocli --deploy-cgi=myapp.me --skip-etc-hosts
```

### Generate virtual host config to standard output

If you want to generate virtual host configuration without actually modifying
web server configuration, you can use `--stdout` command line option.
This option will generate virtual host configuration  and print it to standard output. It is useful if you want to deploy configuration manually.

Because it will not change any web server configuration, you do not need to run it with root privilege. So following code is suffice.

```
$ fanocli --deploy-cgi=myapp.me --stdout
```

## Apache

This section explains how to deploy web application as CGI application on Apache web server.

### Dedicated server

This section explains how to deploy server that you have full control and administrative privilege (aka root access).

#### Debian-based Linux

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

Above snippet basically tells Apache to allow (`+ExecCGI`) to serve any file ends with `cgi` extension as CGI script (`AddHandler cgi-script .cgi`). `-MultiViews` also important to disable automatic content negotiation because we need to add URL rewriting to allow use of routing in our application. So our application will be responsible to decide what response to return.
You need to enable `mod_rewrite` module, otherwise, routing will not work.
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

You need to make sure that `app.cgi` has execution bit. Read *Executable binary file permission* in [Deployment](/deployment) for more information.

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

- Reload Apache service

```
$ sudo service apache2 reload
```

If configuraton is ok, Apache will load new configuration.

- Setup domain name to DNS

If you have not register a domain name with domain name registrar, you can setup fake domain name for local development only by adding
entry to `/etc/hosts` file. Open `/etc/hosts` and add following entry,

```
127.0.0.1 fano.dev
```

`127.0.0.1` with assumption that Apache run on same machine.

#### Fedora-based Linux

The difference between Debian and Fedora regarding Apache mostly about default service name (Debian uses `apache2` and Fedora uses `httpd`), user which Apache run (Debian uses `www-data` user while Fedora uses `apache`) and default Apache configuration location.

- Create virtual host

Apache main configuration is stored in `/etc/httpd/httpd.conf`. To add virtual host configuration, you can simply add entry to this file or prefered way is to create configuration in file `/etc/httpd/conf.d/` directory.

In Fedora and Apache 2.4, `/etc/httpd/conf.d/` directory will be search for additional configurations. So you can just create virtual host file inside this directory.

- Reload Apache service

Tell Apache to load configuration by running

```
$ sudo systemctl reload httpd
```

### Shared-hosting

This section explains how to deploy server that you have no full control and have very limited administrative privilege.

What you need to do is to copy executable binary into `cgi-bin` and any public files to `public_html` or `public`. You may want to make sure that relative path works correctly.

## Nginx

Nginx does not support running CGI program, only FastCGI. So either you create [FastCGI program](/scaffolding-with-fano-cli/creating-project/#scaffolding-fastcgi-project) or use tools such [fcgiwrap](https://www.nginx.com/resources/wiki/start/topics/examples/fcgiwrap/) that allows you to run CGI program as FastCGI program.

## Simulate run on command line

If you use Fano CLI to generate CGI web application, it will add
`tools/simulate.run.sh` bash script. It can be used to simplify simulating run application in shell.

For example,

```
$ ./tools/simulate.run.sh
```

or to change route to access, set `REQUEST_URI` variable.

```
$ REQUEST_URI=/test/test ./simulate.run.sh
```

or more elaborate call,

```
$ cd app/public
$ REQUEST_METHOD=GET \
  REQUEST_URI=/test/test \
  SERVER_NAME=juhara.com \
  ./app.cgi
```

This is similar to simulating browser requesting this page,for example,

```
$ wget -O- http://[your fano app hostname]/test/test
```

However, running using `tools/simulate.run.sh` allows you to view output of `heaptrc` unit for detecting memory leak (if you enable `-gh` switch in `build.dev.cfg`).

## Explore more

- [Deployment](/deployment)
- [Scaffolding with Fano CLI](/scaffolding-with-fano-cli)
