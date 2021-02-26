---
title: Creating HTTPS web application
description: Step by step tutorial for creating basic HTTPS web application
---

<h1 class="major">Creating HTTPS web application</h1>

In this step by step tutorial, we are going to create a simple HTTPS web application using Fano CLI. Read [Scaffolding with Fano CLI](/scaffolding-with-fano-cli) for more information.

## Who is this tutorial for?

This tutorial is written for anyone new to Fano Framework. It is assumed that you have proficiency with Free Pascal and its language syntax. You are familiar with  Linux command and and its shell terminal.

## Requirement

- [Free Pascal >= 3.0](https://www.freepascal.org)
- [git](https://git-scm.com/)
- Text editor, vim, VSCode, Atom etc.
- [Fano CLI](https://github.com/fanoframework/fano-cli)
- Working Internet connection.

## Create Project

Make sure that all requirements are met. To create a HTTP project using `libmicrohttpd`, run

```
$ fanocli --project-mhd=myhttps.fano
```

Wait until new project is created. If everything is ok, your project will be created inside `myhttps.fano` directory. Change active directory to `myhttps.fano` directory.

```
$ cd myhttps.fano
```

All actions below are assumed to take place inside `myhttps.fano` directory.

## Create controller

Create main controller

```
$ fanocli --controller=Home --route=/
```
## Setup SSL certificate
For our development  purpose we only need to use self-signed certificate. If you plan to have a publicly reachable server, you will need to ask a trusted third party, called *Certificate Authority*, to attest the certificate for you.

We need to create private key. We use OpenSSL to generate 1024 bit key. For our purpose, 1024 bit key is sufficient.

```
openssl genrsa -out myhttps.key 1024
```

If everything is ok, you should find `myhttps.key` file in current directory. Next we need to generate certificate using private key. Following command generate x509 certificate valid for one year.

```
openssl req -days 365 -out myhttps.pem -new -x509 -key myhttps.key
```
You will be asked serveral questions, make sure `Common Name` match URI of your domain.
If command is succesful, you should find `myhttps.pem` file in current directory.

## Add certificate in application

Open `src/app.pas` and edit lines like so

```
svrConfig.useTLS := true;
svrConfig.tlsKey := getCurrentDir() + '/myhttps.key';
svrConfig.tlsCert := getCurrentDir() + '/myhttps.pem';
```
## Build application

Run

```
$ ./build.sh
```

Wait until compilation is finished. If you get error such as

```
/usr/bin/ld : cannot find -lmicrohttpd
```

You need to [install libmicrohttpd development package](/known-issues#missing-libmicrohttpd-development-package) first. Run `build.sh` after you install it.

## Run application

Run

```
$ ./bin/app.cgi
```

## Access application from browser

By default, our application listens on port `20477`. Open web browser and go to `https://127.0.0.1:20477`. Internet browser will show warning. This is because we use self-signed certificate. Just ignore it and you should see text *Home Controller* is printed on browser.

## Summary

In this tutorial, you have learned how to use Fano CLI to

- Setup web application project directory with Fano Framework,
- Create controller,
- Create self-signed SSL certificate.
- Add SSL certificate to application.

## Explore more

- [Step by Step Tutorials](/tutorials)
- [Scaffolding with Fano CLI](/scaffolding-with-fano-cli)

