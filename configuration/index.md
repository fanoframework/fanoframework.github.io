---
title: Application Configuration
description: Tutorial on how to work with configuration
---

<h1 class="major">Configuration</h1>

## About configuration

There are times when you need to avoid hard coding value in application because
you need to change it without having to rebuild application binary. For example, you need to be able to store database credential in configuration rather than hard coding, so you can change it easily.

Fano Framework provides `IAppConfiguration` interface for that purpose.

## IAppConfiguration

`IAppConfiguration` interface provides several methods,

- `getString()` accepts name and returns string value.
- `getInt()` accepts name returns integer value.
- `getBool()` accepts name returns boolean value.
- `getFloat()` accepts name returns double value.
- `has()` test if name is exists in configuration.

## Built-in IAppConfiguration implementation

Fano Framework provides `TJsonFileConfig`, `TIniFileConfig`, `TEnvConfig` class which loads configuration data from JSON, INI file and environment variables respectively.

Fano Framework also provides `TCompositeConfig` and `TNullConfig` class. First one is `IAppConfiguration` implementation with capability to combine two`IAppConfiguration` instance and latter is null class implements `IAppConfiguration` interface.

### Load config from JSON

```
var config : IAppConfiguration;
...
config := TJsonFileConfig.create(
    getCurrentDir() + '/config/config.json'
);
```

### Load config from INI

```
var config : IAppConfiguration;
...
config := TIniFileConfig.create(
    getCurrentDir() + '/config/config.ini',
    'fano'
);
```
Last parameter is name of default section to use. Read [INI file configuration](#ini-file-configuration) section in this document for more information.

### Load configuration from environment variables

```
var config : IAppConfiguration;
...
config := TEnvConfig.create();
```

### Combine multiple configurations as one

`TCompositeConfig` allows you to use multiple configurations. In following setup,
if configuration not found in environment variables, then it will try to find it in
config.json.

```
var config : IAppConfiguration;
...
config := TCompositeConfig.create(
    TEnvConfig.create(),
    TJsonFileConfig.create(
        getCurrentDir() + '/config/config.json'
    )
);
```

## Register config instance to dependency container

To be able to use `TJsonFileConfig`, `TIniFileConfig`, `TEnvConfig`, `TCompositeConfig` and `TNullConfig` class with [dependency container](/dependency-container), Fano Framework provides `TJsonFileConfigFactory`, `TIniFileConfigFactory`, `TEnvConfigFactory`, `TCompositeConfigFactory` and `TNullConfigFactory` class which enables you to register above classes in container.

### Register JSON config
```
container.add(
    'config',
    TJsonFileConfigFactory.create(
        getCurrentDir() + '/config/config.json'
    )
);
```

### Register INI config
```
container.add(
    'config',
    TIniFileConfigFactory.create(
        getCurrentDir() + '/config/config.ini'
    )
);
```

### Register environment variable config
```
container.add(
    'config',
    TEnvConfigFactory.create()
);
```

### Register composite configurations
```
container.add(
    'config',
    TCompositeConfigFactory.create(
        TNullConfigFactory.create(),
        TJsonFileConfigFactory.create(getCurrentDir() + '/config/config.json')
    )
);
```

### Retrieve configuration instance from dependency container

To get configuration instance

```
var config : IAppConfiguration;
...
config := container.get('config') as IAppConfiguration;
```
or with array-like syntax

```
config := container['config'] as IAppConfiguration;
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

`getString()`, `getInt()`, `getBool()` and `getFloat()` methods accept second parameter
if you want to use different value when key does not exist.

```
baseUrl := config.getString('baseUrl', 'https://example.com');
```
If key `baseUrl` is not found, it returns second parameter value.


## <a name="ini-file-configuration"></a>INI file configuration

`TIniFileConfig` is thin wrapper of Free Pascal `TIniFile`. `TIniFile` cannot read data from INI file that has no section. Your INI file must contain at least one section which serve as default section. The last parameter of `TIniFileConfig`'s constructor expect name of default section. If you use `TIniFileConfigFactory`, by default it uses `fano` as default section if not specified. You can specify default section by calling `setDefaultSection()` method as shown in following code.

```
container.add(
    'config',
    TIniFileConfigFactory.create(
        getCurrentDir() + '/config/config.ini'
    ).setDefaultSection('hello')
);
```

Consider reading `cookie.maxAge` configuration in code example above. It will read `maxAge` from `cookie` section.

```
[cookie]
maxAge=3600
```

However, because nested section are not allowed in INI file, you can only read one section. For example,

```
nestedData := config.getInt('fano.data.nested');
```
will read data from

```
[fano]
data.nested=test
```

## Setting up application configuration with Fano CLI
When creating new project, you can use `--config` parameter to setup application configuration.
Read [Setup application configuration when creating project](/scaffolding-with-fano-cli/creating-project#setup-application-configuration-when-creating-project) for more information.

You can also manually setting application configuration by overriding `buildAppConfig()` method of `TBasicAppServiceProvider` or `TDaemonAppServiceProvider` class as shown in following code sample.

```
TAppServiceProvider = class(TDaemonAppServiceProvider)
protected
    function buildAppConfig(
        const container : IDependencyContainer
    ) : IAppConfiguration; override;
    ...
end;

...

function TAppServiceProvider.buildAppConfig(
    const container : IDependencyContainer
) : IAppConfiguration;
begin
    container.add(
        'config',
        TIniFileConfigFactory.create(
            getCurrentDir() + '/config/config.ini'
        )
    );
    result := container['config'] as IAppConfiguration;
end;

```

## Explore more

- [Dependency Container](/dependency-container)
- [Working with Application](/working-with-application)
