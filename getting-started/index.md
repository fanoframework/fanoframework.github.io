---
title: Getting Started
description: Basic tutorial to get started with Fano Framework, web application framework for modern Pascal programming language
---

<h1 class="major">Getting Started</h1>

Following document explains minimum steps required to setup a working Fano web application on development machine.

## Requirement

- Linux
- [Free Pascal >= 3.0](https://www.freepascal.org)
- [git](https://git-scm.com/)
- [Apache 2.4](https://httpd.apache.org/)
- [Fano CLI](https://github.com/fanoframework/fano-cli)
- Working Internet connection.
- Root privilege (for setting up virtual host).

## Install Fano CLI

Install [Fano CLI](https://github.com/fanoframework/fano-cli) as described in [Installation](/scaffolding-with-fano-cli#installation) section of [Scaffolding with Fano CLI](/scaffolding-with-fano-cli) documentation.

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

## Project directory
Fano CLI creates several files and directories.

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
- Home controller `src/App/Home/Controllers/HomeController.pas` is code that prints `Home controller` text.
- Include file `src/Routes/Home/route.inc` associates default URL `/` with home controller.
- Include file `src/Dependencies/controllers.dependencies.inc` registers factory class for home controller.

## Explore more

- [Step by Step Tutorials](/tutorials)
- [Examples](/examples)
- [Scaffolding with Fano CLI](/scaffolding-with-fano-cli)
- [Creating Hello World application with Fano CLI](https://fanoframework.github.io/tutorials/hello-world-application-with-fano-cli)
