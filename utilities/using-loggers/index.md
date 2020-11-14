---
title: Using Loggers
description: How to use loggers in Fano Framework
---
<h1 class="major">Using Loggers</h1>

## Logger and why should you care?

There are times when developers need to log something to keep track of things, e.g,
what went wrong (a.k.a error logs), auditing purpose and so on.

Fano Framework provides logging mechanism thorough `ILogger` interface. This interface exposes several methods:

- `log()` to log messages in any levels.
- `emergency()` to log emergency messages.
- `critical()` to log critical messages.
- `error()` to log error messages.
- `alert()` to log alert messages.
- `warning()` to log warning messages.
- `debug()` to log debug messages.
- `notice()` to log notice messages.
- `info()` to log information messages.

Except `log()` method, all methods expect two parameter, first parameter is message to log and second parameter is data related to log message. This parameter is optional. If this second parameter is given, then it must implements
`ISerializeable` interface.

- [See ILogger source code](https://github.com/fanoframework/fano/blob/master/src/Libs/Logger/Contracts/LoggerIntf.pas) for more information.
- [ISerializeable interface](https://github.com/fanoframework/fano/blob/master/src/Core/Contracts/SerializeableIntf.pas)

For example to log critical message,

```
var logger : ILogger;
...
logger.critical('This is critical message');
```

## Built-in logger

Fano Framework has several built-in loggers that developers can use or extend.
All built-in loggers inherit from `TAbstractLogger` which provides most of method implementations except `log()` method which is an abstract method that descendant needs to implements. This class is mostly what developers need to extends in order to create new type of logger.

### Log to system log

`TSysLogLogger` is logger implementation which write log to system log. See [syslog(3) man page](http://man7.org/linux/man-pages/man3/syslog.3.html).

```
var logger : ILogger;
...
logger := TSysLogLogger.create();
```

Its constructor expects three optional parameters.

```
constructor TSysLogLogger.create(
    const prefix : string = '';
    const opt : integer = -1;
    const facility : integer = -1
);
```

- `prefix`, string that is prefixed to all log messages,
- `opt`, integer value for option. If not set, then by default it use `LOG_NOWAIT or LOG_PID`.
- `facility`, integer value for facility. If not set, then by default it use `LOG_USER`.

See [openlog(3) man page](http://man7.org/linux/man-pages/man3/openlog.3.html) for more information.

### Log to file

`TFileLogger` is logger implementation which write log to file. Its constructor expects filename where to write log. `EInOutError` exception will be triggered when
filename can not be created or open for writing. In that case, make sure correct
directory/file permission is given.

```
var logger : ILogger;
...
logger := TFileLogger.create('storages/logs/app.log');
```

### Suppress logging

`TNullLogger` is logger implementation that does nothing. It is provided so developer can disable logging.

### Logging to STDOUT

`TStdOutLogger` is logger implementation that will output log message to STDOUT.

### Logging to STDERR

`TStdErrLogger` is logger implementation that will output log message to STDERR.

### Logging to Database

`TDbLogger` is logger implementation that will output log message to database. It requires instance of `IRdbms` which responsible to do database operation. Read [Database](/database) section for more information.

To be able to use this logger, you need to create a table with at least four columns to store level, message, log datetime and associated data related to log entry. For example,

```
CREATE TABLE logs
(
    id INT PRIMARY KEY NOT NULL AUTO_INCREMENT,
    level VARCHAR(20) NOT NULL,
    msg VARCHAR(400) NOT NULL,
    created_at DATETIME NOT NULL,
    context TEXT NOT NULL
)
```
When create `TDbLogger` instance, you need to tell about table name  and required fields,

```
logger := TDbLogger.create(
    rdbms,
    'logs', //table name
    'level', //level column name
    'msg', //message column name
    'created_at', //log datetime
    'context' //context column name
);
```

### Log to several medium

`TCompositeLogger` is logger implementation that is composed from external `ILogger` instances. It can be used to combine two or more loggers as one, thus creating more complex logging functionality. For example, to log to file and to log to RDBMS simultaneously.

```
var logger : ILogger;
...
logger := TCompositeLogger.create([
    TFileLogger.create('storages/logs/app.log'),
    TSomeLoggerThatLogToDb.create()
]);
```

### Separate log based on level type

Sometime you may want to separate log based on level. For example, you may want
to log non critical log message to separate file than critical one for easier
audit. Or you need to disable logging  for non critical log message but keep critical log.

`TSegregatedLogger` is logger implementation which suitable for this kind of logging scenario. This logger expects caller to provide four `ILogger` instances
which will handle `INFO`, `DEBUG`, `WARNING` and `CRITICAL` log level respectively.

```
var logger : ILogger;
...
logger := TSegregatedLogger.create(
    TFileLogger.create('storages/logs/app.emergency.log'),
    TFileLogger.create('storages/logs/app.alert.log'),
    TFileLogger.create('storages/logs/app.critical.log'),
    TFileLogger.create('storages/logs/app.error.log'),
    TFileLogger.create('storages/logs/app.warning.log'),
    TFileLogger.create('storages/logs/app.notice.log'),
    TFileLogger.create('storages/logs/app.info.log'),
    TFileLogger.create('storages/logs/app.debug.log')
);
```
This way, each log type is written in separate file, thus make it easier to find,
for example, critical log messages.

Should you need to disable logging `INFO` level message, you can replace with something like follows

```
var logger : ILogger;
...
logger := TSegregatedLogger.create(
    TFileLogger.create('storages/logs/app.emergency.log'),
    TFileLogger.create('storages/logs/app.alert.log'),
    TFileLogger.create('storages/logs/app.critical.log'),
    TFileLogger.create('storages/logs/app.error.log'),
    TFileLogger.create('storages/logs/app.warning.log'),
    TFileLogger.create('storages/logs/app.notice.log'),
    TNullLogger.create(),
    TFileLogger.create('storages/logs/app.debug.log')
);
```

Or you may want to combine `INFO`, `DEBUG` into one file and `WARNING`, `CRITICAL` log message as other file.

```
var logger, infoDebug, warningCritical : ILogger;
...
infoDebug := TFileLogger.create('storages/logs/app.infodebug.log'),
warningCritical := TFileLogger.create('storages/logs/app.warningcritical.log'),
logger := TSegregatedLogger.create(
    TFileLogger.create('storages/logs/app.emergency.log'),
    TFileLogger.create('storages/logs/app.alert.log'),
    warningCritical,
    TFileLogger.create('storages/logs/app.error.log'),
    warningCritical,
    TFileLogger.create('storages/logs/app.notice.log'),
    infoDebug,
    infoDebug
);
```

## Factory class for built-in loggers

Fano Framework provides some factory class for built-in loggers so they can
be registered in [dependency container](/dependency-container) easily. Following factories are available:

- `TFileLoggerFactory`, factory class for `TFileLogger`.
- `TNullLoggerFactory`, factory class for `TNullLogger`.
- `TCompositeLoggerFactory`, factory class for `TCompositeLogger`.
- `TSegregatedLoggerFactory`, factory class for `TSegregatedLogger`.
- `TStdOutLoggerFactory`, factory class for `TStdOutLogger`.
- `TStdErrLoggerFactory`, factory class for `TStdErrLogger`.
- `TSysLogLoggerFactory`, factory class for `TSysLogLogger`.
- `TDbLoggerFactory`, factory class for `TDbLogger`.

### Register TFileLogger

For example, to register `TFileLogger` in dependency container:

```
var container : IDependencyContainer;
...
container.add('logger', TFileLoggerFactory.create('storages/logs/app.log'));
```

### Register TNullLogger

To register `TNullLogger`,

```
container.add('logger', TNullLoggerFactory.create());
```

### Register TStdErrLogger

To register `TStdErrLogger`,

```
container.add('logger', TStdErrLoggerFactory.create());
```

### Register TCompositeLogger

To register `TCompositeLogger`,

```
container.add(
    'logger',
    TCompositeLoggerFactory.create(
        TFileLoggerFactory.create('storages/logs/app.log'),
        TNullLoggerFactory.create()
    )
);
```
### Register TSysLogLogger

To register `TSysLogLogger`,

```
container.add(
    'logger',
    (TSysLogLoggerFactory.create()).prefix('fano-app')
);
```
### Register TDbLogger

To register `TDbLogger`,

```
container.add('db', TMySqlDbFactory.create(
    'mysql 5.7',
    '127.0.0.1',
    '[replace with database name]',
    '[replace with username]',
    '[replace with password]',
    3306
));
container.add('logger', TDbLoggerFactory.create().rdbmsSvcName('db'));
```
`rdbmsSvcName()` is used to tell factory where to look for `IRdbms`  instance.

Code above assume using table name `logs`, with column `level`, `msg`, `created_at` and `context`.

To use different table name or column names,
```
container.add(
    'logger',
    TDbLoggerFactory.create()
        .rdbmsSvcName('db')
        .tableName('mylogs')
        .levelField('mylevel')
        .msgField('mymsg')
        .createdAtField('my_created_at')
        .contextField('my_context')
);
```
### Retrieve ILogger instance from container

After logger factory is registered, you can access logger anywhere from application as shown in following code.

```
var logger : ILogger;
...
logger := container.get('logger') as ILogger;
```
or with array-like syntax

```
logger := container['logger'] as ILogger;
```

## Explore more

- [Error Handler](/error-handler)
- [Database](/database)
- [Example web application that log messages to MySQL database](https://github.com/fanoframework/fano-db-logger)
