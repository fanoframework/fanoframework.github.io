---
title: Scaffolding with Fano CLI
description: Tutorial how to use Fano CLI to scaffold web application using Fano Framework
---
<h1 class="major">Scaffolding with Fano CLI</h1>

## What is Fano CLI?

[Fano CLI](https://github.com/fanoframework/fano-cli) is command line application
to help scaffolding project structure using [Fano Framework](https://github.com/fanoframework/fano).

## Installation

Follow installation instruction in Fano CLI [README.md](https://github.com/fanoframework/fano-cli/blob/master/README.md) document.

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

This command line options creates Git repository and initial commit for you  automatically. This behavior may cause problem if you already create remote repository and try to merge local repository with remote one. Git may refuse
to merge because they have unrelated commit histories.

To workaround this problem, you can run `git merge` with option `--allow-unrelated-histories` or create project directory without creating initial commit or create project directory structure without
Git repository.

## Scaffolding FastCGI project directory structure

To scaffold FastCGI project structure using Fano framework, run with  `--project-fcgi` command line options

```
$ fanocli --project-fcgi=[project-name]
```

For example, following command will cause a new FastCGI project created in directory name `test-fano-fcgi` inside current directory.

```
$ fanocli --project-fcgi=test-fano-fcgi
```

Generated project files are mostly similar to `--project` output, except that `src/app.pas`, and `src/bootstrap.pas` which will generate a daemon FastCGI web application.
See [Deploy as FastCGI application](/deployment/fastcgi) for information on how to
setup FastCGI application to work with various web server.

## Scaffolding SCGI project directory structure

To scaffold SCGI project structure using Fano framework, run with  `--project-scgi` command line options

```
$ fanocli --create-project-scgi=[project-name]
```

For example, following command will cause a new SCGI project created in directory name `test-fano-scgi` inside current directory.

```
$ fanocli --create-project-scgi=test-fano-scgi
```

Generated project files are mostly similar to `--project-fcgi` output but for SCGI protocol. See [Deploy as SCGI application](/deployment/scgi) for information on how to
setup SCGI application to work with various web server.

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

## Creating controller

After you create project structure, to scaffold controller class, run with  `--controller` command line options

```
$ cd test-fano
$ fanocli --controller=Hello
```

It will create following files

```
test-fano/app/App/Hello/Controllers/HelloController.pas
test-fano/app/App/Hello/Controllers/Factories/HelloControllerFactory.pas
test-fano/app/Routes/Hello/routes.inc
```

and also modify following files

```
test-fano/app/bootstrap.pas
test-fano/app/Dependencies/controllers.dependencies.inc
test-fano/app/Routes/routes.inc
test-fano/build.cfg
test-fano/build.cfg.sample
```

By default, it will create GET route to `/hello`. So after running `build.sh`
script, you can access `HelloController` by visiting URL

```
wget http://[your host name]/hello
```

## Creating view

After you create project structure, to scaffold view class, run with  `--view` command line options

```
$ cd test-fano
$ fanocli --view=Hello
```

It will create following files

```
test-fano/app/App/Hello/Views/HelloView.pas
test-fano/app/App/Hello/Views/Factories/HelloViewFactory.pas
```

and also modify following files

```
test-fano/app/bootstrap.pas
test-fano/app/Dependencies/views.dependencies.inc
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
test-fano/app/App/Hello/Models/HelloModel.pas
test-fano/app/App/Hello/Models/Factories/HelloModelFactory.pas
```

and also modify following files

```
test-fano/app/bootstrap.pas
test-fano/app/Dependencies/models.dependencies.inc
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

## Deployment

Fano CLI offers commands to simplify setting up Fano web application with various web server through following commands,

- `--deploy-cgi`, setup Fano web application as CGI with a web server.
- `--deploy-fcgi`, setup Fano web application as FastCGI with a web server.
- `--deploy-fcgid`, setup Fano web application as FastCGI with a Apache web server and mod_fcgid.
- `--deploy-scgi` , setup Fano web application as SCGI with a Apache web server.

All commands above requires root privilege, so you need to run it with `sudo`.

```
$ sudo fanocli --deploy-cgi=myapp.com
```

Read [Deployment](/deployment) for more information on how to use commands above.

## Build

Change active directory to new project directory after you created project and run

```
$ /build.sh
```

to build application.

## Explore more

- [Deployment](/deployment)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
