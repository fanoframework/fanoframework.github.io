---
title: Creating Hello World FastCGI web application with Fano CLI
description: Step by step tutorial for creating basic hello world FastCGI web application with Fano CLI
---

<h1 class="major">Creating Hello World FastCGI web application With Fano CLI</h1>

In this step by step tutorial, we are going to create a simple Fast CGI web application which displays greeting message using Fano CLI. Read [Scaffolding with Fano CLI](/scaffolding-with-fano-cli) for more information.

## Who is this tutorial for?

This tutorial is written for anyone new to Fano Framework. It is assumed that you have proficiency with Free Pascal and its language syntax. You are familiar with  Linux command and and its shell terminal.

## Requirement

- [Free Pascal >= 3.0](https://www.freepascal.org)
- [git](https://git-scm.com/)
- [Apache 2.4](https://httpd.apache.org/)
- [Apache mod_fcgid](https://httpd.apache.org/mod_fcgid/mod/mod_fcgid.html)
- Text editor, vim, VSCode, Atom etc.
- [Fano CLI](https://github.com/fanoframework/fano-cli)
- Working Internet connection.
- Root privilege.

## Create Project

Make sure that all requirements are met. To create a FastCGI project, run

```
$ fanocli --project-fcgid=Hello
```

Wait until new project is created. If everything is ok, your project will be created inside `Hello` directory. Change active directory to `Hello` directory.

```
$ cd Hello
```

All actions below is assumed to take place inside `Hello` directory.

## Build application

Run

```
$ ./build.sh
```

Wait until compilation is finished.

## Setup web server configuration

Run

```
$ sudo fanocli --deploy-fcgid=hello.fano
```

## Run it from browser

Open web browser and go to `http://hello.fano`. You should see *Fano Application Error* page. It is normal as you have not create any route to handle the request.

## Create a controller

Run

```
$ fanocli --controller=Home --route=/
```

Command above will create `src/App/Home/Controllers/HomeController.pas` and register route `/` so you will be able to access it via URL `http://hello.fano/`. Please note that, without `--route` parameter, by default Fano CLI register route `/home`.

Rebuild application by running,

```
$ ./build.sh
```

After compilation is finished, open web browser and go to `http://hello.fano`. You should see text `Home controller` printed. Congratulations, your application is working.

## Summary

In this tutorial, you have learned how to use Fano CLI to

- Setup FastCGI web application project directory with Fano Framework,
- Deploy application with Apache web server,
- Create controller,
- Create default route.

## Explore more

- [Step by Step Tutorials](/tutorials)
- [Scaffolding with Fano CLI](/scaffolding-with-fano-cli)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
