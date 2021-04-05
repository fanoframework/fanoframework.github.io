---
title: Getting Started
description: Basic tutorial to get started with Fano Framework, web application framework for modern Pascal programming language
---

<h1 class="major">Getting Started</h1>

Following document explains minimum steps required to setup a working Fano web application on development machine.

## Requirement

- Linux or FreeBSD
- [Free Pascal >= 3.0](https://www.freepascal.org)
- [git](https://git-scm.com/)
- [Apache 2.4](https://httpd.apache.org/)
- [mod_cgi](https://httpd.apache.org/docs/current/mod/mod_cgi.html) or [mod_cgid](https://httpd.apache.org/docs/current/mod/mod_cgid.html)
- [mod_rewrite](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)
- [Fano CLI](https://github.com/fanoframework/fano-cli)
- Working Internet connection.
- Root privilege (for setting up virtual host).

## Install Fano CLI

Install [Fano CLI](https://github.com/fanoframework/fano-cli) as described in *[Installation](/scaffolding-with-fano-cli#installation)* section of *[Scaffolding with Fano CLI](/scaffolding-with-fano-cli)* documentation.

## Create application

Make sure all requirements above are met. Run

```
$ fanocli --project-cgi=Hello
$ cd Hello
$ fanocli --controller=Home --route=/
$ ./build.sh
$ sudo fanocli --deploy-cgi=hello.fano
```

## Run it from browser

Open web browser and go to `http://hello.fano`. You should see `Home controller` text is printed in browser. Congratulations, your application is working.

Following video shows step by step creating Fano Framework web application project.

[![Getting Started with Fano Framework video](https://img.youtube.com/vi/393pMHp8yj8/sddefault.jpg)](https://www.youtube.com/watch?v=393pMHp8yj8 "Getting Started with Fano Framework video")

## Command walkthrough

Let us take a look at each command above to understand what it does.

Following command tells Fano CLI to create CGI web application project in `Hello` directory. Directory must not exist. Read *[Creating Project with Fano CLI](/scaffolding-with-fano-cli/creating-project)* for creating different web application project ([FastCGI](/scaffolding-with-fano-cli/creating-project#scaffolding-fastcgi-project), [SCGI](/scaffolding-with-fano-cli/creating-project#scaffolding-scgi-project), [uwsgi](/scaffolding-with-fano-cli/creating-project#scaffolding-uwsgi-project) or [http](/scaffolding-with-fano-cli/creating-project#scaffolding-libmicrohttpd-project)).

```
$ fanocli --project-cgi=Hello
```
We change active directory to newly created `Hello` directory.
```
$ cd Hello
```

Create controller name `HomeController.pas` that will handle request to route `/`. For more information regarding route, read *[Working with Router](/working-with-router)*.
```
$ fanocli --controller=Home --route=/
```
Without `--route=/`, by default, Fano CLI will create route same as lower case of controller's name, i.e, `/home`. If you omit `--route=/` then you can only access `HomeController` using URL `http://hello.fano/home` instead of `http://hello.fano`.

Compile application
```
$ ./build.sh
```

Setup a virtual host for domain `hello.fano` and associate it with our CGI application binary.
```
$ sudo fanocli --deploy-cgi=hello.fano
```
`--deploy-cgi` modifies web server configuration and reload it. That is why you need to use `sudo` command. Read *[Deployment](/deployment)* for deploying different protocol web application project ([FastCGI](/deployment/fastcgi), [SCGI](/deployment/scgi), [uwsgi](/deployment/uwsgi), [http](/deployment/standalone-web-server)).

## Project directory walkthrough
Fano Framework has no opinion about your project directory structure.
You can structure your project directories and files the way you like.
However, Fano CLI creates several files and directories that follows certain assumptions.

### Directories
- `src`, application project source code directory
- `bin`, compiled binaries output directory,
- `config`, application configuration directory
- `resources`, application resources directory, such as HTML templates, SCSS, etc.
- `storages`, application runtime-generated files directory, such as session files, logs etc.
- `tools`, helper shell scripts directory, such as scripts to clean compiled binaries.
- `public`, application document root directory where public resources resides such as images, cascade stylesheets, JavaScripts files.

### Files

- Main program source code is `src/app.pas`.
- Main unit `src/bootstrap.pas` glues all modules.
- Home controller `HomeController.pas` in `src/App/Home/Controllers` is code that prints `Home controller` text.
- Include file `src/Routes/Home/route.inc` associates default URL `/` with home controller.
- Include file `controllers.dependencies.inc` in `src/Dependencies` directory, registers factory class for home controller.

## Explore more

- [Step by Step Tutorials](/tutorials)
- [Examples](/examples)
- [Scaffolding with Fano CLI](/scaffolding-with-fano-cli)
- [Creating Hello World application with Fano CLI](https://fanoframework.github.io/tutorials/hello-world-application-with-fano-cli)
