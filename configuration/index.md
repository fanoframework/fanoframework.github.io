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

`IAppConfiguration` interface provides two methods

- `getString()` which accepts name and returns string value
- `getInt()` which accept name returns integer value
- `getBool()` which accept name returns boolean value

## Built-in IAppConfiguration implementation

Fano Framework provides `TJsonFileConfig` and `TIniFileConfig` class which loads configuration data from JSON and INI file respectively. Also available `TNullConfig` which is null class implements `IAppConfiguration` interface.

Load config from JSON,

```
var config : IAppConfiguration;
...
config := TJsonFileConfig.create(
    getCurrentDir() + '/config/config.json'
);
```

Load config from INI,

```
var config : IAppConfiguration;
...
config := TIniFileConfig.create(
    getCurrentDir() + '/config/config.ini',
    'fano'
);
```
Last parameter is name of default section to use. Read [INI file configuration](#ini-file-configuration) section in this document for more information.

To be able to use `TJsonFileConfig` and `TIniFileConfig` class with [dependency container](/dependency-container), Fano Framework provides `TJsonFileConfigFactory` and `TIniFileConfigFactory` class which enables you to register above classes in container.

Register JSON config
```
container.add(
    'config',
    TJsonFileConfigFactory.create(
        getCurrentDir() + '/config/config.json'
    )
);
```

Register INI config
```
container.add(
    'config',
    TIniFileConfigFactory.create(
        getCurrentDir() + '/config/config.ini'
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
Read [Setup application configuration when creating project](/scaffolding-with-fano-cli#setup-application-configuration-when-creating-project) for more information.

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
