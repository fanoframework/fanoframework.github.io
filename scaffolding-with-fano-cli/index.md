---
title: Scaffolding with Fano CLI
description: Tutorial how to use Fano CLI to scaffold web application using Fano Framework
---
<h1 class="major">Scaffolding with Fano CLI</h1>

## What is Fano CLI?

[Fano CLI](https://github.com/fanoframework/fano-cli) is command line application
to help scaffolding project structure using [Fano Framework](https://github.com/fanoframework/fano). It helps tedious tasks such as
[creating Fano web application project](#creating-project), [creating controller](#creating-controller), [model](#creating-model), [view](#creating-view), [middleware](#creating-middleware) classes,
[generate random key](#generate-random-key) and [GUID](#generate-guid), [minify JavaScript files](#minify-javascript) and also [setting up web server configuration](#deployment).

## <a name="installation"></a>Installation

Make sure Free Pascal and git are installed. Run

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

To view available commands, you can run

```
$ fanocli --help
```

## <a name="creating-project"></a>Creating Web Application Project

Fano CLI provides several commands for scaffolding Fano Framework web application easily, such as, `--project-cgi`, `--project-fcgi`, `--project-fcgid`, `--project-scgi`,  `--project-uwsgi` and `--project-mhd` which is to create web application project using CGI, FastCGI, SCGI and uwsgi, http (using [libmicrohttpd](https://www.gnu.org/software/libmicrohttpd/)) protocol.

```
$ fanocli --project-cgi=[project-name]
```

`[project-name]` is directory where project resides and will be created by Fano CLI. Please read [Creating Project with Fano CLI](/scaffolding-with-fano-cli/creating-project) for more detail explanation of each command.


## <a name="creating-controller"></a>Creating controller

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

To create controller for certain route pattern or HTTP method, add `--route` and `--method` parameter. For example, to create controller which will handle POST request to `/my/hello`,

```
$ fanocli --controller=Hello --route=/my/hello --method=POST
```

## <a name="creating-view"></a>Creating view

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

## <a name="creating-model"></a>Creating model

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

## <a name="add-middleware-support"></a>Add middleware support

Any [project creation commands](/scaffolding-with-fano-cli/creating-project), i.e, `--project*` commands, accept additional parameter `--with-middleware`. If it is set, then during project creation, [middleware support](/middlewares) is added to generated project.

```
$ fanocli --project-cgi=Hello --with-middleware
```

## <a name="creating-middleware"></a>Creating middleware

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

## <a name="generate-random-key"></a>Generate random key

To generate random key (for example, encryption secret key), run with  `--key=[length]` command line options

```
$ fanocli --key=32
```

If `length` is not set, it is assumed 64 bytes of random value. Output is Base64 encoded string of random bytes.
In unix or Linux, it reads from `/dev/urandom`. In Windows, it uses CryptoAPI.

## <a name="generate-guid"></a>Generate GUID

To generate GUID, run with  `--guid` command line options

```
$ fanocli --guid
```

## <a name="minify-javascript"></a> Minify JavaScript files

To reduce JavaScript file size, run with  `--jsmin=[path]` command line options.

With `[path]` empty, it will scan current directory for any JavaScript files and minify them and output as new file with name using original file name appended with `min.js`.

```
$ fanocli --jsmin
```

With `[path]` not empty, if it is directory, it will scan that directory for any JavaScript files and minify them and output as new file with name using original file name appended with `min.js`. If `[path]` is single file, it minify it and create new file with same name appended with `min.js`.

```
$ fanocli --jsmin=/path/to/js
```

If you specify optional `--output=[target-path]` parameter, then `[target-path]` will be used as its filename.

```
$ fanocli --jsmin=/path/to/js --output=/path/to/js/scripts.min.js
```

If `[target-path]` equals `stdout`, minify output is printed to STDOUT.

```
$ fanocli --jsmin=/path/to/js --output=stdout
```

## <a name="add-session-support"></a>Add session support

Any [project creation commands](/scaffolding-with-fano-cli/creating-project), i.e, `--project*` commands, accept additional parameter `--with-session=[session storage]` where `session storage` value can be one of following value

- `file`, create session which stores session data in file.
- `cookie`, create session which stores session data in encrypted cookie.
- `db`, create session which stores session data in database. This is not implemented yet.

If `[session storage]` not set then it is assumed `file`. So following commands are identical

```
$ fanocli --project-cgi=Hello --with-session=file
$ fanocli --project-cgi=Hello --with-session
```

For `file` and `cookie`, you can define format of session data with `--type` parameter which expect value of `json` or `ini` for JSON or INI format respectively.

For example, following command will cause session support to be added to project and each session data will be stored as INI file.

```
$ fanocli --project-cgi=Hello --with-session=file --type=ini
```

Because in Fano Framework, session support is implemented with middleware infrastructure, `--with-session` implies usage of `--with-middleware` so following command are identical

```
$ fanocli --project-cgi=Hello --with-session
$ fanocli --project-cgi=Hello --with-session --with-middleware
```

Please read [Working with Session](/working-with-session) for more information about session.

## <a name="deployment"></a>Deployment

Fano CLI offers commands to simplify setting up Fano web application with various web server through following commands,

- `--deploy-cgi`, setup Fano CGI web application with a web server.
- `--deploy-fcgi`, setup Fano FastCGI web application with a web server as reverse proxy.
- `--deploy-fcgid`, setup Fano FastCGI web application with a Apache web server and mod_fcgid.
- `--deploy-scgi` , setup Fano SCGI web application with a web server as reverse proxy.
- `--deploy-lb-scgi` , setup Fano SCGI web application with a web server  as reverse proxy and load balancer.
- `--deploy-lb-fcgi` , setup Fano FastCGI web application with a web server  as reverse proxy and load balancer.
- `--deploy-uwsgi` , setup Fano uwsgi web application with a web server as reverse proxy.
- `--deploy-lb-uwsgi` , setup Fano uwsgi web application with a web server as reverse proxy and load balancer.
- `--deploy-http` , setup Fano http web application with a web server as reverse proxy.
- `--deploy-lb-http` , setup Fano http web application with a web server as reverse proxy and load balancer.

All commands above requires root privilege, so you need to run it with `sudo`.

```
$ sudo fanocli --deploy-cgi=myapp.com
```

Developer is responsible to make sure that deployment matched project type. For example, project created with `--project-fcgi` can only be deployed with `--deploy-fcgi` or `--deploy-lb-fcgi`.

Read [Deployment](/deployment) for more information on how to use commands above.

## Setup daemon as service in systemd

In Linux distribution which use systemd, such as latest Fedora or Debian, Fano CLI provides command `--daemon-sysd=[service name]` to simplify registering daemon web application with systemd.

```
$ sudo fanocli --daemon-sysd=hello
```

After that, you can run application using following command,

```
$ sudo systemctl start hello
```

or to stop it,

```
$ sudo systemctl stop hello
```

`--daemon-sysd` command accept additional parameter,

- `--user=[username]`, username whose application is run as. If not set, then current non-root user is used.
- `--bin=[application binary path]`, absolute path to application executable binary. If not set then it is assumed absolute path of `bin/app.cgi`.

```
$ sudo fanocli --daemon-sysd=hello --user=fano --bin=/home/my.fano/bin/myapp.bin
```

## Build

Change active directory to new project directory after you created project and run

```
$ /build.sh
```

to build application.

## Explore more

- [Deployment](/deployment)
- [Creating Hello World web application With Fano CLI](/tutorials/hello-world-application-with-fano-cli)
