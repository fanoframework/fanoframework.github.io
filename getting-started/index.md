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

To update Fano to its latest commit, run

```
$ git checkout master && git submodule foreach --recursive git pull origin master
```

Above command will checkout to `master` branch of this repository and pull latest update from `master` branch of [Fano](https://github.com/fanoframework/fano) repository.

### Copy compiler configuration

Copy `*.cfg.sample` to `*.cfg`.
Make adjustment as you need in `build.cfg`, `build.prod.cfg`, `build.dev.cfg`
and run `build.sh` shell script (if you are on Windows, then `build.cmd`).

These `*.cfg` files contain some Free Pascal compiler switches that you can turn on/off to change how executable is compiled and generated. For complete
explanation on available compiler switches, consult Free Pascal documentation.

Also copy `app/config/config.json.sample` to `app/config/config.json` and edit
configuration as needed. For example, you may need to change `baseUrl` to match your own base url so JavaScript or CSS stylesheets point to correct URL.

```
$ cp app/config/config.json.sample app/config/config.json
$ cp build.prod.cfg.sample build.prod.cfg
$ cp build.dev.cfg.sample build.dev.cfg
$ cp build.cfg.sample build.cfg
```

`tools/config.setup.sh` shell script is provided to simplify copying those
configuration files. Following shell command is similar to command above.

```
$ ./tools/config.setup.sh
```

## Build Application

Fano App repository comes with several shell scripts to simplify task. The most important is `build.sh` (or `build.cmd` on Windows)

After you configure `*.cfg`, to compile and build binary executable, run

```
$ ./build.sh
```

By default, it will output binary executable in `app/public` directory.


### Build for different environment

To build for different environment, set `BUILD_TYPE` environment variable.

#### Build for production environment

    $ BUILD_TYPE=prod ./build.sh

Build process will use compiler configuration defined in `fano/fano.cfg`, `build.cfg` and `build.prod.cfg`. By default, `build.prod.cfg` contains some compiler switches that will aggressively optimize executable both in speed and size.

#### Build for development environment

```
$ BUILD_TYPE=dev ./build.sh
```

Build process will use compiler configuration defined in `fano/fano.cfg`, `build.cfg` and `build.dev.cfg`.

If `BUILD_TYPE` environment variable is not set, production environment will be assumed.

### Change executable output directory

Compilation will output executable to directory defined in `EXEC_OUTPUT_DIR`
environment variable. By default is `app/public` directory.

```
$ EXEC_OUTPUT_DIR=/path/to/public/dir ./build.sh
```

### Change executable name

Compilation will use executable filename as defined in `EXEC_OUTPUT_NAME`
environment variable. By default is `app.cgi` filename.

```
$ EXEC_OUTPUT_NAME=index.cgi ./build.sh
```

## Run Application

### Run with a webserver

Setup a virtual host. Please consult documentation of web server you use.

For example on Apache,

```
<VirtualHost *:80>
     ServerName www.example.com
     DocumentRoot /home/example/app/public

     <Directory "/home/example/app/public">
         Options +ExecCGI
         AllowOverride FileInfo
         Require all granted
         DirectoryIndex app.cgi
         AddHandler cgi-script .cgi
     </Directory>
</VirtualHost>
```
On Apache, you will need to enable CGI module, such as `mod_cgi` or `mod_cgid`. If CGI module not loaded, above virtual host will cause `app.cgi` is downloaded instead of executed.

For example, on Debian, this will enable `mod_cgi` module.

```
$ sudo a2enmod cgi
$ sudo systemctl restart apache2
```

Depending on your server setup, for example, if  you use `.htaccess`, add following code:

```
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^(.*)$ app.cgi [L]
</IfModule>
```
and put `.htaccess` file in same directory as `app.cgi` file (i.e., in `app/public` directory).

Content of `.htaccess` basically tells Apache to serve existing files/directories directly. For any non-existing files/directories, pass them to our application.

Fano App repository contains sample `app/public/htaccess.example` file that you can use. Just copy and rename it to `.htaccess`.

After you setup virtual host, try open application through Internet browser.

### Simulate run on command line

```
$ cd app/public
$ REQUEST_METHOD=GET \
  REQUEST_URI=/test/test \
  SERVER_NAME=juhara.com \
  ./app.cgi
```

`tools/simulate.run.sh` is bash script that can be used to simplify simulating run
application in shell.

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
