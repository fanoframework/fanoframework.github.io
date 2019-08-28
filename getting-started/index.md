---
title: Getting Started
description: Basic tutorial to get started with Fano Framework, web application framework for modern Pascal programming language
---

<h1 class="major">Getting Started</h1>

## Requirement

Main requirement is [Free Pascal](https://www.freepascal.org/) >= 3.0

From your shell run

```
$ fpc -i
```

If you see following similar output

```
Free Pascal Compiler version 3.0.4

Compiler date      : 2017/10/03
Compiler CPU target: x86_64
...
```
then you are good. If not, you may not have Free Pascal installed in your system.
In that case, you may want to take a look following resources:

- [Download Free Pascal](https://www.freepascal.org/download.var)
- [Free Pascal Documentation](https://www.freepascal.org/docs.var)

## Installation

### Clone from Fano App example web application

```
$ git clone git@github.com:fanoframework/fano-app.git --recursive
```

`--recursive` is needed so git also pull [Fano](https://github.com/fanoframework/fano) repository.

If you are missing `--recursive` when you clone, you may find that `fano` directory is empty. In this case run

```
$ git submodule update --init
```

## Copy compiler configuration

Run

```
$ ./tools/config.setup.sh
```

It will setup required configuration files.

## Build Application

Run

```
$ ./build.sh
```

By default, it will output binary executable in `app/public` directory.

## Run Application

### Run with a webserver

Setup a virtual host. Please consult documentation of web server you use.

For example on Apache,

```
<VirtualHost *:80>
     ServerName example.fano
     DocumentRoot /home/example/app/public

     <Directory "/home/example/app/public">
         Options +ExecCGI
         AllowOverride FileInfo
         Require all granted
         DirectoryIndex app.cgi
         AddHandler cgi-script .cgi

        <IfModule mod_rewrite.c>
            RewriteEngine On
            RewriteCond %{REQUEST_FILENAME} !-d
            RewriteCond %{REQUEST_FILENAME} !-f
            RewriteRule ^(.*)$ app.cgi [L]
        </IfModule>
     </Directory>


</VirtualHost>
```
On Apache, you will need to enable CGI module, such as `mod_cgi` or `mod_cgid`. If CGI module not loaded, above virtual host will cause `app.cgi` is downloaded instead of executed.

Content of `<IfModule>` above basically tells Apache to serve existing files/directories directly. For any non-existing files/directories, pass them to our application.

For example, on Debian, this will enable `mod_cgi` module.

```
$ sudo a2enmod cgi
$ sudo systemctl restart apache2
```

Setup domain name for example by adding DNS record or for development purpose add
follwing entry to `/etc/hosts`

```
127.0.0.1 example.fano
```
After you setup virtual host and domain name, try open application through Internet browser with URL `http://example.fano`.

### Simulate run on command line

```
$ cd app/public
$ REQUEST_METHOD=GET \
  REQUEST_URI=/test/test \
  SERVER_NAME=juhara.com \
  ./app.cgi
```

`tools/simulate.run.sh` is bash script that can be used to simplify simulating run application in shell.

```
$ ./tools/simulate.run.sh
```

or to change route to access, set `REQUEST_URI` variable.

```
$ REQUEST_URI=/test/test ./simulate.run.sh
```

This is similar to simulating browser requesting this page,for example,

```
$ wget -O- http://[your fano app hostname]/test/test
```

However, running using `tools/simulate.run.sh` allows you to view output of `heaptrc`
unit for detecting memory leak (if you enable `-gh` switch in `build.dev.cfg`).


## Deployment

You need to deploy only executable binary and any supporting files such as HTML templates, images, css stylesheets, application config.
Any `pas` or `inc` files or shell scripts is not needed in deployment machine in order application to run.

So for this repository, you will need to copy `public`, `Templates`, `config`
and `storages` directories to your deployment machine. make sure that
`storages` directory is writable by web server.

Read [Deployment](/deployment) for information how to deploy Fano Framework web
application on various web server setup.

## Explore more

- [Step by Step Tutorials](/tutorials)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
