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
- Text editor, vim, VSCode, Atom etc.
- [Fano CLI](https://github.com/fanoframework/fano-cli)
- Working Internet connection.
- Root privilege (for setting up virtual host).

## Install Fano CLI

Install [Fano CLI](https://github.com/fanoframework/fano-cli) as described in [Installation](/scaffolding-with-fano-cli#installation) section of [Scaffolding with Fano CLI](/scaffolding-with-fano-cli).

## Create application

Make sure all requirements above are met. Run

```
$ fanocli --project=Hello
$ cd Hello
$ fanocli --controller=Home --route=/
$ ./build.sh
$ sudo fanocli --deploy-cgi=hello.fano
```

## Run it from browser

Open web browser and go to `http://hello.fano`. You should see `Home controller` text is printed in browser. Congratulations, your application is working.

## Explore more

- [Step by Step Tutorials](/tutorials)
- [Examples](/examples)
- [Scaffolding with Fano CLI](/scaffolding-with-fano-cli)
- [Creating Hello World application with Fano CLI](https://fanoframework.github.io/tutorials/hello-world-application-with-fano-cli)
