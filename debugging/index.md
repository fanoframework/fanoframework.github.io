---
title: Debugging
description: Debugging Fano Framework web application
---

<h1 class="major">Debugging</h1>

This section explains how to debug Fano Framework web application. This document assumes that you know how to create Fano Framework web application project. If you do not, you should read [Getting Started](/getting-started) document first.

## Debugging in Visual Studio Code IDE

### Install NativeDebug extension

We will install [Native Debug](https://github.com/WebFreak001/code-debug) extension to debug in Visual Studio Code with GDB debugger. Installation process is straightforward. Press `Ctrl + Shift + X` keyboard shortcut to open *Extensions* tab. Type *Native Debug* to search extension and click *Install* button. Wait until installation is complete.

### Create launch.json file

Open *Run* tab (`Ctrl + Shift + D`) click *Run and Debug* button and select GDB environment.
Visual Studio Code will create `launch.json` file. Replace `name` and `target` like so

```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug Fano",
            "type": "gdb",
            "request": "launch",
            "target": "./bin/app.cgi",
            "cwd": "${workspaceRoot}",
            "valuesFormatting": "parseText"
        }
    ]
}
```
Save and close it.

Now when you open *Run* tab (`Ctrl + Shift + D`) you will see *Debug Fano* on top.

### Build executable for development

To be able to debug, we need debugging information inside executable binary. Go to
shell and from inside project root directory, run

```
$ ./tools/clean.sh
$ BUILD_TYPE=dev ./build.sh
```

This will compile Fano Framework web application with debugging information.

### Start debugging

Back to Visual Studio Code IDE. Open source code file you want to debug and toggle breakpoint at line of interest by clicking on empty area left of line number. Activate *Run* tab click green button on left of *Debug Fano* to start debugging.

This will run application. Open Internet browser and type URL so that breakpoint is hit.

## Debugging in Lazarus IDE

*TODO*

## Debugging CGI web application

It may be tricky to debug CGI web applications as they are run by web server. However you can simulate running CGI script using `tools/simulate.run.sh` shell script that is generated when you use [Fano CLI](/scaffolding-with-fano-cli) to create project.

To debug CGI web application with Visual Studio Code, edit `launch.json`

```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug Fano CGI",
            "type": "gdb",
            "request": "launch",
            "target": "./public/app.cgi",
            "env": {
                "REQUEST_METHOD": "GET",
                "REQUEST_URI" : "/some-url",
                "QUERY_STRING" : "order=123&user_id=12"
            }
            "cwd": "${workspaceRoot}",
            "valuesFormatting": "parseText"
        }
    ]
}
```

This will simulate run CGI web application when open from Internat browser via URL
`http://your.app/some-url?order=123&user_id=12`.

## Explore more

- [Getting started](/getting-started)
- [Step-By-Step Tutorials](/tutorials)
