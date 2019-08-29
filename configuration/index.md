---
title: Application Configuration
description: Tutorial on how to work with configuration
---

<h1 class="major">Configuration</h1>

## About configuration

There are times when you need to avoid hard coding value in application because
you need to change it without having to rebuild application binary.

For example, you need to be able to store database credential in configuration rather than hard coding, so you can change it easily.

Fano Framework provides `IAppConfiguration` interface for that purpose.

## IAppConfiguration

`IAppConfiguration` interface provides two methods

- `getString()` which accepts name and returns string value
- `getInt()` which accept name returns integer value

## Built-in IAppConfiguration implementation

Fano Framework provides `TJsonFileConfig` class which loads configuration data from
JSON file.

```
var config : IAppConfiguration;
...
config := TJsonFileConfig.create(
    getCurrentDir() + '/config/config.json'
);
```

To be able to use `TJsonFileConfig` class with [dependency container](/dependency-container), Fano Framework provides `TJsonFileConfigFactory` class which enables you to register `TJsonFileConfig` class in container.

```
container.add(
    'config',
    TJsonFileConfigFactory.create(
        getCurrentDir() + '/config/config.json'
    )
);
```

To get configuration instance

```
var config : IAppConfiguration;
...
config := container.get('config') as IAppConfiguration;
```

## Read configuration data

Suppose you have JSON file with content as follows

```
{
    "appName" : "MyApp",

    "baseUrl" : "http://myapp.fano",

    "session" : {
        "name" : "fano_sess",
        "dir" : "storages/sessions/"
    },

    "cookie" : {
        "domain" : "myapp.fano",
        "maxAge" : 3600
    }
}
```

To get `baseUrl` value,

```
var baseUrl : string;
...
baseUrl := config.getString('baseUrl');
```

To get `dir` value,

```
var sessionDir : integer;
...
sessionDir := config.getString('session.dir');
```

To get `maxAge` value,

```
var cookieMaxAge : integer;
...
cookieMaxAge := config.getInt('cookie.maxAge');
```

## Explore more

- [Dependency Container](/dependency-container)
- [Working with Application](/working-with-application)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
