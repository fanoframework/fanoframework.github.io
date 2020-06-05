---
title: Example applications
description: List of example application built with Fano Framework
---

<h1 class="major">Example Application</h1>

Below is list of example applications which you can use to get started. They are
generated with [Fano CLI](https://github.com/fanoframework/fano-cli), command line tools to help [scaffolding web application](/scaffolding-with-fano-cli) with Fano Framework.

## Hello World applications

List of getting started web application for various supported protocol, i.e [CGI](https://tools.ietf.org/html/rfc3875), [FastCGI](http://www.mit.edu/~yandros/doc/specs/fcgi-spec.html), [SCGI](http://python.ca/scgi/protocol.txt) and [uwsgi](https://uwsgi-docs.readthedocs.io/en/latest/Protocol.html).

- [Basic CGI web application example](https://github.com/fanoframework/fano-app), [live example](https://fano.juhara.id/)
- [Fano Fcgid](https://github.com/fanoframework/fano-fcgid)
- [Example FastCGI/CGI web application which can be deployed as FastCGI with mod_fcgid or as CGI](https://github.com/fanoframework/fano-cgi-fcgi)
- [Example FastCGI web application listen on Unix domain socket](https://github.com/fanoframework/fano-fcgi-unix)
- [Example FastCGI web application listen on TCP port](https://github.com/fanoframework/fano-fastcgi)
- [Example web application using scgi protocol](https://github.com/fanoframework/fano-scgi)
- [Example web application using uwsgi protocol](https://github.com/fanoframework/fano-uwsgi)
- [Example web application using http protocol](https://github.com/fanoframework/fano-http)

## MIME types web application

Following example applications show how to [work with response](/working-with-response) other than HTML page, such as image, PDF and JSON data.

- [Image generator CGI web application](https://github.com/fanoframework/fano-app-img) shows how to output image that is generated on-the fly, See [live example here](https://fano-img.juhara.id/)
- [Fano Api](https://github.com/fanoframework/fano-api) demonstrates how to return JSON response.
- [Fano Pdf](https://github.com/fanoframework/fano-pdf) demonstrates how to generate PDF document on the fly.

## MVC web application

[Fano Mvc](https://github.com/fanoframework/fano-mvc) demonstrate how to separate logic of application using [Model](/working-with-models), [View](/working-with-views), [Controller](/working-with-controllers). It also demonstrates how to use HTML view template to compose application UI layout.

## File upload

[Handling upload example](https://github.com/fanoframework/fano-upload) shows how to [handle file upload](/handling-file-upload) in Fano Framework.
For example how to validate file upload, read [Form Validation](#form-validation) section below.

## Database

Following example applications show how to work with SQL and NoSQL databases by modelling data as [model](/working-with-models).

- [Example web application that load data from Elasticsearch](https://github.com/fanoframework/fano-elasticsearch)
- [Example SCGI web application that load data from Elasticsearch](https://github.com/fanoframework/fano-elastic)
- [MySQL web application example](https://github.com/fanoframework/fano-app-db)
- [Database connection pool example](https://github.com/fanoframework/fano-db-pool)
- [PostgreSQL web application example](https://github.com/fanoframework/fano-postgresql)
- [Example web application that log messages to MySQL database](https://github.com/fanoframework/fano-db-logger). This example demonstrates [logging functionality](/utilities/using-loggers) in Fano Framework.

## Middleware

[Example web application using middleware](https://github.com/fanoframework/fano-app-middleware) demonstrates [how to use middleware](/middlewares) to protect one or more application [routes](/working-with-router).

## Working with Session

Following example applications demonstrate how to use session with Fano Framework.

- [Session example](https://github.com/fanoframework/fano-session) demonstrates how to use session that is stored in file on server.
- [Session in encrypted cookie example](https://github.com/fanoframework/fano-session-cookie) demonstrates session that is stored in encrypted cookie.

## Cross-Origin Resource Sharing (CORS)

[Cross-Origin Resource Sharing (CORS) example](https://github.com/fanoframework/fano-cors) demonstrates how to [add CORS headers](/security/handling-cors) to web application.

## Cross-Site Request Forgery (CSRF) protection

[Cross-Site Request Forgery (CSRF) protection example](https://github.com/fanoframework/fano-csrf) demonstrate how to [protect application from CSRF attack](/security/csrf-protection).

## <a name="form-validation"></a>Form validation

- [Form Validation example](https://github.com/fanoframework/fano-validation) demonstrate how to use Fano Framework input [validation feature](/security/form-validation).
- [File upload validation example](https://github.com/fanoframework/fano-scgi-upload) demonstrate how to use Fano Framework input [validation feature](/security/form-validation) to validate [file upload](/handling-file-upload) using various validation rules.

## HTTP Verb tunneling examples

[HTTP verb tunnelling example application](https://github.com/fanoframework/fano-verb-tunneling) demonstrate how to use [HTTP verb tunneling](/security/http-verb-tunneling) when application behind strict firewall policy.

## HTTP Authentication examples

[HTTP Basic Authentication example application](https://github.com/fanoframework/fano-basic-auth) demonstrates how to use [HTTP Basic Authentication](/security/http-authentication) (RFC 2617) in Fano Framework.

## Handling request with JSON body

[Fano Json Request](https://github.com/fanoframework/fano-json-request) demonstrates how to [handle request with JSON body](/working-with-request#handling-request-with-json-body).

## HTTP cache header examples

[Fano Cache Control](https://github.com/fanoframework/fano-cache-control) demonstrates how to [add HTTP cache header](/working-with-response/http-cache-header).

## Send email examples

[Fano Email](https://github.com/fanoframework/fano-email) demonstrates how to [send email](/utilities/sending-email).

## Password hash examples

[Fano Password example](https://github.com/fanoframework/fano-password) demonstrates how to [verify plain password against password hash](/security/password-hash).
It is similar to session-related examples above but it compares password using password hash verification.

## Explore more

- [Getting started](/getting-started)
- [Step-By-Step Tutorials](/tutorials)
