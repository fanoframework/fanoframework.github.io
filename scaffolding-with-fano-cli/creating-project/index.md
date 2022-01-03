---
title: Creating Project with Fano CLI
description: Tutorial how to use Fano CLI to scaffold web application project using Fano Framework
---
<h1 class="major">Creating Project with Fano CLI</h1>

## <a name="scaffolding-cgi-project"></a>Scaffolding CGI project directory structure

To scaffold project structure using Fano framework, run with  `--project-cgi` command line options

```
$ fanocli --project-cgi=[project-name]
```

For example, following command will cause a new project created in directory name `test-fano` inside current directory.

```
$ fanocli --project-cgi=test-fano
```

## <a name="scaffolding-fcgid-project"></a>Scaffolding FastCGI project directory structure with Apache mod_fcgid module

To scaffold FastCGI project structure using Fano framework that employ Apache mod_fcgid, run with `--project-fcgid` command line options

```
$ fanocli --project-fcgid=[project-name]
```

For example, following command will cause a new FastCGI project created in directory name `test-fano-fcgi` inside current directory.

```
$ fanocli --project-fcgid=test-fano-fcgi
```

See [Deploy as FastCGI application](/deployment/fastcgi) for information on how to
setup FastCGI application to work with various web server.

## <a name="scaffolding-fastcgi-project"></a>Scaffolding FastCGI project directory structure

To scaffold FastCGI project structure using Fano framework, run with  `--project-fcgi` command line options

```
$ fanocli --project-fcgi=[project-name]
```

For example, following command will cause a new FastCGI project created in directory name `test-fano-fcgi` inside current directory.

```
$ fanocli --project-fcgi=test-fano-fcgi
```

Additionally, you can set `--host` and `--port` where FastCGI will listen. If omitted, `127.0.0.1` and `20477` will be used as host and port respectively.

```
$ fanocli --project-fcgi=test-fano-fcgi --host=192.168.2.1 --port=4040
```

Generated project files are mostly similar to `--project-cgi` output, except that `src/app.pas`, and `src/bootstrap.pas` which will generate a daemon FastCGI web application.
See [Deploy as FastCGI application](/deployment/fastcgi) for information on how to
setup FastCGI application to work with various web server.

Difference between FastCGI application created with `--project-fcgid` and `--project-fcgi`, first command will create FastCGI application which its process is managed by Apache mod_fcgid module while latter must be run independently using Apache mod_proxy_fcgi or Nginx with reverse proxy.

## <a name="scaffolding-scgi-project"></a>Scaffolding SCGI project directory structure

To scaffold SCGI project structure using Fano framework, run with  `--project-scgi` command line options

```
$ fanocli --project-scgi=[project-name]
```

For example, following command will cause a new SCGI project created in directory name `test-fano-scgi` inside current directory.

```
$ fanocli --project-scgi=test-fano-scgi
```

Additionally, you can set `--host` and `--port` where SCGI will listen. If omitted, `127.0.0.1` and `20477` will be used as host and port respectively.

```
$ fanocli --project-scgi=test-fano-scgi --host=192.168.2.1 --port=4040
```

Generated project files are mostly similar to `--project-fcgi` output but for SCGI protocol. See [Deploy as SCGI application](/deployment/scgi) for information on how to
setup SCGI application to work with various web server.

## <a name="scaffolding-uwsgi-project"></a>Scaffolding uwsgi project directory structure

To create web application that use [uwsgi protocol](https://uwsgi-docs.readthedocs.io/en/latest/Protocol.html), use `--project-uwsgi` parameter. Other parameters are similar to SCGI or FastCGI project above.

See [Deploy as uwsgi application](/deployment/uwsgi) for information on how to
setup uwsgi application to work with various web server.

## <a name="scaffolding-libmicrohttpd-project"></a>Scaffolding libmicrohttpd http project directory structure

To create web application that use http protocol using [libmicrohttpd](https://www.gnu.org/software/libmicrohttpd/), use `--project-mhd` parameter. Other parameters are similar to SCGI, FastCGI or uwsgi project above.

It will add conditional define `-dLIBMICROHTTPD` in `build.cfg` which cause libmicrohttpd to be linked with application.

You need to [install libmicrohttpd development package](/known-issues#missing-libmicrohttpd-development-package) otherwise you get linking error.

See [Deploy as standalone web server](/deployment/standalone-web-server) for information on how to setup http application to work as a standalone web server or run behind various reverse proxy web server.

### Add https support
Inside generated project directory, you can find that `src/app.pas` will have following commented code

```
//svrConfig.useTLS := true;
//svrConfig.tlsKey := '/path/to/ssl/cert/key';
//svrConfig.tlsCert := '/path/to/ssl/cert/cert';
```
If you want to support https, you need to uncomment those three lines of codes and set correct path for SSL certificate files. You need to make sure that they are readable by application.

## <a name="scaffolding-http-project"></a>Scaffolding http project directory structure

To create http web application that use Free Pascal built-in http server,`TFpHttpServer` class, use `--project-http` parameter. Other parameters are similar to SCGI, FastCGI or uwsgi project above.

See [Deploy as standalone web server](/deployment/standalone-web-server) for information on how to setup http application to work as a standalone web server or run behind various reverse proxy web server.

## <a name="scaffolding-http-indy-project"></a>Scaffolding http project with Indy

To create http web application that use Indy http server,`TIdHttpServer` class, use `--project-indy` parameter. Other parameters are similar to SCGI, FastCGI or uwsgi project above.

Make sure you set `INDY_DIR` environment variable to point to where Indy directory installation before compilation.

```
$ export INDY_DIR="/path/to/indy"
```
or you can set it in `.env` file which will be loaded when you run build script. Please note that you should not add trailing slash, so use

```
$ export INDY_DIR="/path/to/indy"
```
and not

```
$ export INDY_DIR="/path/to/indy/"
```

When you use Indy, you must aware of [Indy memory leak issue](/known-issues#indy-memory-leak-issue).

See [Deploy as standalone web server](/deployment/standalone-web-server) for information on how to setup http application to work as a standalone web server or run behind various reverse proxy web server.

## <a name="use-fano-framework-specific-release-version"></a>Use Fano Framework specific release version

By default, when you use any `--project-*` option, it will use latest commit of `master` branch of Fano Framework repository. To use Fano Framework specific release version when creating project, add `--fano-ver=[VER]` option, `[VER]` is string that contains specific release tag to use. For example, to use Fano Framework version 1.0.0, run following command.

```
$ fanocli --project-scgi=example.fano --fano-ver=v1.0.0
```
Visit [Fano Framework releases page](https://github.com/fanoframework/fano/releases) for complete list of release versions.

## <a name="setup-application-configuration-when-creating-project"></a>Setup application configuration when creating project

All commands for creating project, for example, `--project-cgi`, `--project-fcgi`, `--project-scgi`, etc, accept additional parameter `--config=[configType]` where `[configType]` is either `ini` or `json`.

If `--config` is set, then during project creation, it generates application configuration files and register application configuration to [dependency container](/dependency-container). If `--config` is set but its value is empty string, `json` is assumed.

```
$ fanocli --project-fcgi=test-fano --config=ini
```

Read [Application Configuration](/configuration) for more information.

## Add session support

Use `--with-session` to [add session support](/scaffolding-with-fano-cli#add-session-support). Read [Session documentation](/working-with-session) for more information.

```
$ fanocli --project-fcgi=test-fano --config=ini
```

## Add middleware support

Use `--with-middleware` to [add middleware support](/scaffolding-with-fano-cli#add-middleware-support). Read [Middleware documentation](/middlewares) for more information.

## Add CSRF support

Use `--with-csrf` to [add CSRF support](/scaffolding-with-fano-cli#add-csrf-support). Read [Cross-Site Request Forgery (CSRF) protection](/security/csrf-protection) for more information.

## Add logger dependencies

Use `--with-logger` to [add logger dependencies](/scaffolding-with-fano-cli#add-logger-dependencies) to [service container](/dependency-container). Read [Using Loggers](/utilities/using-loggers) for more information.

## Unrelated commit histories issue

`--project-*` command line options creates Git repository and initial commit for you automatically. This behavior may cause problem if you already create remote repository and try to merge local repository with remote one. Git may refuse to merge because they have unrelated commit histories.

To workaround this problem, you can run `git merge` with option `--allow-unrelated-histories`
or create project directory structure without Git repository (`--no-git`).

```
$ fanocli --project-cgi=test-fano --no-git
```

or create project directory with Git repository but without creating initial commit (`--no-initial-commit`)

```
$ fanocli --project-cgi=test-fano --no-initial-commit
```

`--no-git` options is provided to enable you to initialize Git repository manually.

After project directory is constructed, you need to execute following commands,

```
$ cd test-fano
$ git init
$ git submodule add https://github.com/fanoframework/fano.git vendor/fano
$ git add .
$ git commit -m "Initial commit"
```

`--no-initial-commit` is provided to enable you to commit Git repository manually. So you can merge local repository with a remote repository before running git commit. After project directory is constructed, you need to execute following commands,

```
$ cd test-fano
$ git commit -m "Initial commit"
```

## <a name="use-libcurl-in-application"></a>Use libcurl in application
If you want to use curl-based HTTP client such as [`THttpGet` class](https://github.com/fanoframework/fano/blob/master/src/Libs/HttpClient/Implementations/Curl/HttpGetImpl.pas), use `--with-curl` parameter. It adds conditional define `-dLIBCURL` in `defines.cfg` file which causing libcurl library linked with application.

```
$ fanocli --project-scgi=hello --with-curl
```
You need to install libcurl development package otherwise you get [missing libcurl error](/known-issues/#missing-libcurl-development-package).

## Explore more

- [Deployment](/deployment)
- [Scaffolding with Fano CLI](/scaffolding-with-fano-cli)
