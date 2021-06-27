---
title: Documentation
description: Documentation and developer's resources for Fano Framework, web application framework for modern Pascal programming language
---

## Tutorials

- [Getting Started](/getting-started)
- [Step-By-Step Tutorials](/tutorials)
- [Example Applications](/examples)
- [Scaffolding with Fano CLI](/scaffolding-with-fano-cli).

## Application

- **[Working with Application](/working-with-application)**. It explains basic concept of Fano Framework web application.
- **[Dependency Container](/dependency-container)**. Explains dependency injection in Fano Framework.
- **[Dispatcher](/dispatcher)**. Explains basic concept of dispatcher.
- **[CGI Environment](/environment)**. Tutorial on how to read server environmental variables.
- **[Configuration](/configuration)**. How to setup and read application settings.
- **[Error Handler](/error-handler)**. How to handle error in Fano Framework.
- **[Middlewares](/middlewares)**. It explains middleware concept and how to use it in Fano Framework web application.
- **[Working with Router](/working-with-router)**. It explains how to setup application route and associate URL path pattern with code that handle it.

## Request and Response

- **[Working with Request](/working-with-request)**. It explains how to [get query string parameters](/working-with-request#getting-query-parameters), [read request body data](/working-with-request#get-post-put-patch-data), [read cookie](/working-with-request#retrieve-cookies) and [handle request with JSON body](/working-with-request#handling-request-with-json-body).
- **[Working with Response](/working-with-response)**. It explains how to return response in various format such as HTML, JSON or binary response, for example, image and also setting response cookie. Also, it explains how to set response header or redirect to other location or how to [serve static files](/working-with-response/serve-static-files).
- **[Handling File Upload](/handling-file-upload)**. It explains how to work with file upload.
- **[Working with Session](/working-with-session)**. It explains how to manage state between requests.

## MVC

- [Working With Controllers](/working-with-controllers)
- [Working With Views](/working-with-views)
- [Working With Models](/working-with-models)

## Security

- **[Handling CORS](/security/handling-cors)** discusses how to handle *Cross-origin Resource Sharing (CORS)* issue in Fano Framework.
- **[Form Validation](/security/form-validation)**. It explains how to validate form input data.
- **[CSRF Protection](/security/csrf-protection)** discusses how to protect application from *Cross-site request Forgery (CSRF)* attack.
- **[HTTP Verb Tunnelling](/security/http-verb-tunnelling)** explains about how to override HTTP verb such as `PUT`, `PATCH` and `DELETE`.
- **[HTTP Authentication](/security/http-authentication)** discusses authentication using basic, digest, bearer token authentication. It is also related to JSON Web Token.
- [XSS Protection](/security/xss-protection)
- [Clickjacking Protection](/security/clickjacking-protection)
- **[Password hash](/security/password-hash)** explains how to properly hash password using Argon2i, SCrypt, BCrypt password hash algorithm.
- [JSON Web Token (JWT)](/security/jwt)

## Utilities

- [Using Loggers](/utilities/using-loggers)
- [Use HTTP client to call API](/utilities/http-clients)
- [Sending email](/utilities/sending-email)
- [Identifying client user-agent](/utilities/identifying-client-user-agent)
- [Working with regular expression](/utilities/regular-expression)
- [Rate limiting request](/utilities/rate-limit)

## Database

- [Working with Database](/database)

## Deployment

- [Deploy Fano Framework web application on various server setup](/deployment)
- [Deploy as CGI application](/deployment/cgi)
- [Deploy as FastCGI application](/deployment/fastcgi)
- [Deploy as SCGI application](/deployment/scgi)
- [Deploy as uwsgi application](/deployment/uwsgi)
- [Deploy as http application](/deployment/standalone-web-server)
- [Deploy Fano web application with web server as load balancer](/deployment/load-balancer-setup)

## Debugging

- **[Debugging](/debugging)**. It explains how to debug Fano Framework web application with IDE such as Visual Studio Code or Lazarus.

## Known Issue
- [Issue with GNU Linker](/known-issues#issue-with-gnu-linker)
- [Issue with development tools library search path](/known-issues#issue-with-gcc-library-search-path)
- [Issue on FreeBSD](/known-issues#issue-on-freebsd)
- [Issue with Free Pascal 3.2.0](/known-issues#issue-with-free-pascal-3.2.0)
- [Missing libmicrohttpd development package](/known-issues#missing-libmicrohttpd-development-package)
- [Missing libcurl development package](/known-issues#missing-libcurl-development-package)
- [Missing /etc/fpc.cfg](/known-issues#missing-etc-fpc-cfg)
- [Missing MySQL client library](/known-issues#missing-mysql-client-library)
- [Memory leak due to database shutdown](/known-issues#shut-down-database-server-may-cause-memory-leak)
