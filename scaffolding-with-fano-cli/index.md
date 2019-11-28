---
title: Scaffolding with Fano CLI
description: Tutorial how to use Fano CLI to scaffold web application using Fano Framework
---
<h1 class="major">Scaffolding with Fano CLI</h1>

## What is Fano CLI?

[Fano CLI](https://github.com/fanoframework/fano-cli) is command line application
to help scaffolding project structure using [Fano Framework](https://github.com/fanoframework/fano). It helps tedious tasks such as creating Fano web application project, creating controller, model, view, middleware classes and also setting up web server configuration.

## Installation

Make sure Free Pascal and git is installed. Run

```
$ git clone https://github.com/fanoframework/fano-cli.git
$ cd fano-cli
$ ./tools/config.setup.sh
$ ./build.sh
```
For easier typing, copy `bin/out/fanocli` executable binary to globally accessible location, for example

```
$ sudo cp bin/out/fanocli /usr/local/bin/fanocli
```

Follow installation instruction in Fano CLI [README.md](https://github.com/fanoframework/fano-cli/blob/master/README.md) document for more information.

## View Fano CLI Help

To view available command, you can run

```
$ fanocli --help
```

## Scaffolding project directory structure

To scaffold project structure using Fano framework, run with  `--project` command line options

```
$ fanocli --project=[project-name]
```

For example, following command will cause a new project created in directory name `test-fano` inside current directory.

```
$ fanocli --project=test-fano
```

`--project` command is alias of `--project-cgi` and so it creates CGI web application.

This command line options also creates Git repository and initial commit for you  automatically. This behavior may cause problem if you already create remote repository and try to merge local repository with remote one. Git may refuse
to merge because they have unrelated commit histories.

To workaround this problem, you can run `git merge` with option `--allow-unrelated-histories` or create project directory without creating initial commit or create project directory structure without
Git repository.

## Scaffolding FastCGI project directory structure with Apache mod_fcgid module

To scaffold FastCGI project structure using Fano framework that employ Apache mod_fcgid, run with  `--project-fcgid` command line options

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

Generated project files are mostly similar to `--project` output, except that `src/app.pas`, and `src/bootstrap.pas` which will generate a daemon FastCGI web application.
See [Deploy as FastCGI application](/deployment/fastcgi) for information on how to
setup FastCGI application to work with various web server.

Difference between FastCGI application created with `--project-fcgid` and `--project-fcgi` is first will be FastCGI application which its process is managed by Apache mod_fcgid module while latter must be run independently using Apache mod_proxy_fcgi or Nginx.

## <a name="scaffolding-scgi-project">Scaffolding SCGI project directory structure

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

## <a name="scaffolding-uwsgi-project">Scaffolding uwsgi project directory structure

To create web application that use [uwsgi protocol](https://uwsgi-docs.readthedocs.io/en/latest/Protocol.html), use `--project-uwsgi` parameter. Other parameters are similar to SCGI or FastCGI project above.

See [Deploy as uwsgi application](/deployment/uwsgi) for information on how to
setup uwsgi application to work with various web server.

## Scaffolding project directory structure with Git without initial commit

To scaffold project structure using Fano framework with Git repository initialized but without creating initial commit, run with  `--project-no-commit` command line options

```
$ fanocli --project-no-commit=test-fano
```

This command line options is provided to enable you to commit Git repository manually. So you can merge local repository with a remote repository before
running git commit. After project directory is constructed, you need to execute following shell command,

```
$ cd test-fano
$ git commit -m "Initial commit"
```

## Scaffolding project directory structure without Git

To scaffold project structure without initializing
Git repository, run with  `--project-without-git` command line options

```
$ fanocli --project-without-git=test-fano
```

This command line options is provided to enable you to initialize Git repository manually.

After project directory is constructed, you need to execute following shell command,

```
$ cd test-fano
$ git init
$ git submodule add https://github.com/fanoframework/fano.git vendor/fano
$ git add .
$ git commit -m "Initial commit"
```
This command line options is provided to enable you to initialize Git repository manually.

## Setup application configuration when creating project

All command that create project, for example, `--project`, `--project-fcgi`, `--project-scgi`, etc, accept additional parameter `--config=[configType]` where `[configType]` is either `ini` or `json`.

If `--config` is set, then during project creation, it generates application configuration files and register application configuration to [dependency container](/dependency-container). If `--config` is set but its value is empty string, `json` is assumed.

```
$ fanocli --project-fcgi=test-fano --config=ini
```

Read [Application Configuration](/configuration) for more information.

## Creating controller

After you create project structure, to scaffold controller class, run with  `--controller` command line options

```
$ cd test-fano
$ fanocli --controller=Hello
```

It will create following files

```
test-fano/src/App/Hello/Controllers/HelloController.pas
test-fano/src/App/Hello/Controllers/Factories/HelloControllerFactory.pas
test-fano/src/Routes/Hello/routes.inc
```

and also modify following files

```
test-fano/src/bootstrap.pas
test-fano/src/Dependencies/controllers.dependencies.inc
test-fano/src/Routes/routes.inc
test-fano/build.cfg
test-fano/build.cfg.sample
```

By default, it will create GET route to `/hello`. So after running `build.sh`
script, you can access `HelloController` by visiting URL

```
wget http://[your host name]/hello
```

To create controller for certain route pattern or HTTP method, add `--route` and `--method` parameter. For example to create controller which will handle POST request to `/my/hello`,

```
$ fanocli --controller=Hello --route=/my/hello --method=POST
```

## Creating view

After you create project structure, to scaffold view class, run with  `--view` command line options

```
$ cd test-fano
$ fanocli --view=Hello
```

It will create following files

```
test-fano/src/App/Hello/Views/HelloView.pas
test-fano/src/App/Hello/Views/Factories/HelloViewFactory.pas
```

and also modify following files

```
test-fano/src/bootstrap.pas
test-fano/src/Dependencies/views.dependencies.inc
test-fano/build.cfg
test-fano/build.cfg.sample
```

## Creating model

After you create project structure, to scaffold model class, run with  `--model` command line option

```
$ cd test-fano
$ fanocli --model=Hello
```

It will create following files

```
test-fano/src/App/Hello/Models/HelloModel.pas
test-fano/src/App/Hello/Models/Factories/HelloModelFactory.pas
```

and also modify following files

```
test-fano/src/bootstrap.pas
test-fano/src/Dependencies/models.dependencies.inc
test-fano/build.cfg
test-fano/build.cfg.sample
```

## Creating controller, view and model at same time

To simplify creating controller, view and model at same time, you can use
`--mvc` command line option.

```
$ fanocli --mvc=Hello
```

Command above is equal to following command

```
$ fanocli --controller=Hello
$ fanocli --model=Hello
$ fanocli --view=Hello
```

## Creating middleware

To scaffold middleware class, run with  `--middleware` command line options

```
$ cd test-fano
$ fanocli --middleware=AuthOnly
```

It will create following files

```
test-fano/src/Middlewares/AuthOnly/AuthOnlyMiddleware.pas
test-fano/src/Middlewares/AuthOnly/Factories/AuthOnlyMiddlewareFactory.pas
```

## Generate random key

To generate random key, run with  `--key=[length]` command line options

```
$ fanocli --key=32
```

If `length` is not set, it is assumed 64 bytes of random value. Output is Base64 encoded string of random bytes.

## Generate GUID

To generate GUID, run with  `--guid` command line options

```
$ fanocli --guid
```

## Minify JavaScript files

To reduce JavaScript file size, run with  `--jsmin[path]` command line options.

With `[path]` empty, it will scan current directory for any JavaScript files and minify them and output as new file with name using original file name appended with `min.js`.

```
$ fanocli --jsmin
```

With `[path]` not empty. If it is directory, it will scan that directory for any JavaScript files and minify them and output as new file with name using original file name appended with `min.js`. If `[path]` is single file, it minify it and create new file with same name appended with `min.js`.

```
$ fanocli --jsmin=/path/to/js
```

If you specify optional `--output=[target-path]` parameter, then `[target-path]` will be used as its filename.

```
$ fanocli --jsmin=/path/to/js --output=/path/to/js/scripts.min.js
```

If `[target-path]` equals `stdout`, minify output is print to STDOUT.

```
$ fanocli --jsmin=/path/to/js --output=stdout
```

## <a name="add-session-support"></a>Add session support

Any project creation command `--project*` accept additional parameter `--with-session=[session storage]` where `session storage` value can be one of following value

- `file`, create session which stores session data in file.
- `cookie`, create session which stores session data in encrypted cookie.
- `db`, create session which stores session data in database. This is not implemented yet.

```
$ fanocli --project=Hello --with-session=file
```
For `file` and `cookie`, you can define format of session data with `--type` parameter which expect value of `json` or `ini` for JSON or INI format respectively.

For example, following command will cause session support to be added to project and each session data will be stored as JSON file.

```
$ fanocli --project=Hello --with-session=file --type=ini
```
Please read [Working with Session](/working-with-session) for more information about session.

## Deployment

Fano CLI offers commands to simplify setting up Fano web application with various web server through following commands,

- `--deploy-cgi`, setup Fano web application as CGI with a web server.
- `--deploy-fcgi`, setup Fano web application as FastCGI with a web server.
- `--deploy-fcgid`, setup Fano web application as FastCGI with a Apache web server and mod_fcgid.
- `--deploy-scgi` , setup Fano web application as SCGI with a web server.
- `--deploy-lb-scgi` , setup Fano web application as SCGI with a web server and load balancer.
- `--deploy-lb-fcgi` , setup Fano web application as FastCGI with a web server and load balancer.

All commands above requires root privilege, so you need to run it with `sudo`.

```
$ sudo fanocli --deploy-cgi=myapp.com
```

Developer is responsible to make sure that deployment matched project type. For example, project created with `--project-fcgi` can only be deployed with `--deploy-fcgi` or `--deploy-lb-fcgi`.

Read [Deployment](/deployment) for more information on how to use commands above.

## Build

Change active directory to new project directory after you created project and run

```
$ /build.sh
```

to build application.

## Explore more

- [Deployment](/deployment)
- [Creating Hello World web application With Fano CLI](/tutorials/hello-world-application-with-fano-cli)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
