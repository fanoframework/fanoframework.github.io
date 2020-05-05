---
title: Creating Project with Fano CLI
description: Tutorial how to use Fano CLI to scaffold web application using Fano Framework
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

You can also add `--no-git` and `no-initial-commit` parameter.

See [Deploy as uwsgi application](/deployment/uwsgi) for information on how to
setup uwsgi application to work with various web server.


## <a name="scaffolding-libmicrohttpd-project"></a>Scaffolding libmicrohttpd project directory structure

To create web application that use http using [libmicrohttpd](https://www.gnu.org/software/libmicrohttpd/), use `--project-mhd` parameter. Other parameters are similar to SCGI, FastCGI or uwsgi project above.

You also need to install libmicrohttpd development package. For Debian-based distribution,

```
$ sudo apt install libmicrohttpd-dev
```

For Fedora-based distribution,

```
$ sudo yum install libmicrohttpd-devel
```

See [Deploy as standalone web server](/deployment/standalone-web-server) for information on how to setup http application to work as a standalone web server or run behind various reverse proxy web server.

## <a name="setup-application-configuration-when-creating-project"></a>Setup application configuration when creating project

All commands for creating project, for example, `--project-cgi`, `--project-fcgi`, `--project-scgi`, etc, accept additional parameter `--config=[configType]` where `[configType]` is either `ini` or `json`.

If `--config` is set, then during project creation, it generates application configuration files and register application configuration to [dependency container](/dependency-container). If `--config` is set but its value is empty string, `json` is assumed.

```
$ fanocli --project-fcgi=test-fano --config=ini
```

Read [Application Configuration](/configuration) for more information.

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
## Explore more

- [Deployment](/deployment)
- [Scaffolding with Fano CLI](/scaffolding-with-fano-cli)
