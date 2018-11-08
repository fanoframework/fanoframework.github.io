---
title: Scaffolding with Fano CLI
description: Tutorial how to use Fano CLI to scaffold web application using Fano Framework
---
<h1 class="major">Scaffolding with Fano CLI</h1>

## What is Fano CLI?

[Fano CLI](https://github.com/fanoframework/fano-cli) is command line application
to help scaffolding project structure using [Fano Framework](https://github.com/fanoframework/fano).

## Build Executable from Source

Follow instruction in Fano CLI README.md document.

## View Fano CLI Help

To view available command, you can run

```
$ fanocli --help
```

## Scaffolding project directory structure

To scaffolding project structure using Fano framework, run with  `--create-project` command line options

```
$ fanocli --create-project=[another project name]
```

For example, following command will cause a new project created in directory name `test-fano` inside current directory.

```
$ fanocli --create-project=test-fano
```

This command line options creates GIT repository and initial commit for you  automatically. This behavior may cause problem if you already create remote repository and try to merge local repository with remote one. Git may refuse
to merge because they have unrelated commit histories.

To workaround this problem, you can run `git merge` with option `--allow-unrelated-histories` or create project directory structure without
GIT repository.

## Scaffolding project directory structure without GIT

To scaffold project structure using Fano framework but without initializing
GIT repository, run with  `--create-project-without-git` command line options

```
$ fanocli --create-project-without-git=test-fano
```

This command line options is provided to enable you to initialize GIT repository manually.
