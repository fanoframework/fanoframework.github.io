---
title: Using loggers
description: How to use loggers in Fano Framework
---

## Logger and why should you care?

There are times when developers need to log something to keep track of things, e.g,
what went wrong (a.k.a error logs), auditing purpose and so on.

Fano Framework provides logging mechanism thorough `ILogger` interface. This interface exposes several methods:

- `log()` to log messages in any levels.
- `critical()` to log critical messages.
- `debug()` to log debug messages.
- `info()` to log information messages.
- `warning()` to log warning messages.

Except `log()` method, all methods expect two parameter, first parameter is message to log and second parameter is data related to log message. This parameter is optional. If this second parameter is given, then it must implements
`ISerializeable` interface.

- [See ILogger source code](https://github.com/fanoframework/fano/blob/master/Libs/Logger/Contracts/LoggerIntf.pas) for more information.
- [ISerializeable interface](https://github.com/fanoframework/fano/blob/master/Core/Contracts/SerializeableIntf.pas)

For example to log critical message,

```
var logger : ILogger;
...
logger.critical('This is critical message');
```


## Built-in logger

Fano Framework has several built-in loggers that developers can use or extend.
All built-in loggers inherit from `TAbstractLogger` which provides most of method implementations except `log()` method which is an abstract method that child needs to implements. This class is mostly what developers need to extends in order to create new type of logger.

### Log to file

`TFileLogger` is logger implementation which write log to file. Its constructor expects filename where to write log. EInOutError exception will be triggered when
filename can not be created or open for writing. In that case, make sure correct
directory/file permission is given.

```
var logger : ILogger;
...
logger := TFileLogger.create('storages/logs/app.log');
```

### Suppress logging

`TNullLogger` is logger implementation that does nothing. It is provided so developer can disable logging.

### Log to several medium

`TCompositeLogger` is logger implementation that is composed from two external `ILogger` instance and provided so developer can combine two or more loggers as one. For example, to log to file and to log to RDBMS simultaneously.

```
var logger : ILogger;
...
logger := TCompositeLogger.create(
    TFileLogger.create('storages/logs/app.log'),
    TSomeLoggerThatLogToDb.create()
);
```

`TCompositeLogger` can only accept two external loggers. To compose more than two loggers, you need to daisy-chain `TCompositeLogger` instances, for example:

```
var logger : ILogger;
...
logger := TCompositeLogger.create(
    TCompositeLogger.create(
        TFileLogger.create('storages/logs/app.log'),
        TSomeLoggerThatLogToDb.create()
    ),
    TSomeMoreLogger.create()
);
```

## Factory class for built-in loggers

Fano Framework provides some factory class for built-in loggers so they can
be registered in [dependency container](/dependency-container) easily. Following factories are available:

- `TFileLoggerFactory`, factory class for `TFileLogger`.
- `TNullLoggerFactory`, factory class for `TNullLogger`.
- `TCompositeLoggerFactory`, factory class for `TCompositeLogger`.

For example, to register `TFileLogger` in dependency container:

```
var container : IDependencyContainer;
...
container.add('logger', TFileLoggerFactory.create('storages/logs/app.log'));
```

To register `TNullLogger`,

```
container.add('logger', TNullLoggerFactory.create());
```

To register `TCompositeLogger`,

```
container.add('logger',
    TCompositeLoggerFactory.create(
        TFileLoggerFactory.create('storages/logs/app.log'),
        TNullLoggerFactory.create()
    )
);
```



