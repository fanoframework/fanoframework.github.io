---
title: Working with Database
description: Working with database in Fano Framework
---

<h1 class="major">Working with Database</h1>

![Database illustration](/assets/images/database.jpg){:class="image fit"}

Photo by [Jan Antonin Kolar](https://unsplash.com/@jankolar?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/database-administrators?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

## IRdbms interface

`IRdbms` interface is just a thin wrapper for Free Pascal SQLdb package. Currently supported RDBMS systems are

- MySQL via `TMysqlDb` class
- PostgreSQL via `TPostgreSqlDb` class
- Firebird via `TFirebirdDb` class,
- SQLite via `TSQLiteDb` class
- Any databases which support ODBC, via `TOdbcDb` class.

## Creating Database Connection

### MySQL

```
var db : IRdbms;
...
db := TMySQLDb.create('MySQL 5.7');
db.connect('localhost', 'your_db_name', 'your_db_username', 'yoursecretpassword', 3306);
```

It will open database connection to MySQL 5.7 server on localhost on port 3306.

Replace `5.7` with your MySQL version, for example `5.5` to connect to MySQL 5.5 database server. Please note that string `MySQL 5.7` is compared case-insentively. So `MySQL 5.7` or `mysql 5.7` are same.

### PostgreSQL

```
var db : IRdbms;
...
db := TPostgreSqlDb.create();
db.connect('localhost', 'your_db_name', 'your_db_username', 'yoursecretpassword', 5432);
```

It will open database connection to PostgreSQL server on localhost on port 5432.

### Firebird

```
var db : IRdbms;
...
db := TFirebirdDb.create();
db.connect('localhost', 'your_db_name', 'your_db_username', 'yoursecretpassword', 3050);
```

It will open database connection to Firebird server on localhost on port 3050.

### SQLite

SQLite is lightweight non client-server database engine. So you can leave host, port, username and password empty. Database name must be set to path where database file is stored.

```
var db : IRdbms;
...
db := TSQLiteDb.create();
db.connect('', 'your_data.db', '', '', 0);
```

It will open database connection which is stored in `your_data.db` file.

### ODBC

If you use database which not yet supported directly by FreePascal sqldb library, you may use ODBC connection.
`TOdbcDb` class is thin wrapper for `TODBCConnection` class which implements `IRdbms` interface.

For example, if you have `/etc/odbcinst.ini` with content as follows

```
[my-mariadb-odbc-driver]
Description = MariaDB Connector/ODBC v.3.0
Driver = /usr/lib/libmaodbc.so
```
and content of `/etc/odbc.ini`

```
[my-app-db]
Description=My App Database
Driver=my-mariadb-odbc-driver
SERVER=localhost
PORT=3306
USER=<your username>
PASSWORD=<your password>
DATABASE=<database name>
```
To connect to database using `my-app-db` DSN, set database parameter with name of DSN
like so
```
var db : IRdbms;
...
db := TOdbcDb.create();
db.connect('', 'my-app-db', '', '', 0);
```
If you want to change value, for example to use different port than what is defined in `/etc/odbc.ini`, just fill port parameter with desired value

```
var db : IRdbms;
...
db := TOdbcDb.create();
db.connect('', 'my-app-db', '', '', 3307);
```

## Registering IRdbms instance in dependency container

You can register `IRdbms` instance in [dependency container](/dependency-container) so that you can access its instance easily.

```
var container : IDependencyContainer;
...
container.add(
    'db',
    TMysqlDbFactory.create(
        'mysql 5.7',
        'localhost',
        'your_db_name',
        'your_db_username',
        'yoursecretpassword',
        3306
    )
);
```
Replace `TMysqlDbFactory` with `TPostgreSqlDbFactory`, `TFirebirdSqlDbFactory`,
`TSQLiteDbFactory`, `TOdbcDbFactory` for Postgresql, Firebird, SQLite database and ODBC respectively.

For `TOdbcDbFactory`, using ODBC with DSN, you can register simply by using its DSN name for example

```
container.add(
    'db',
    TOdbcDbFactory.create()
        .database('my-app-db')
);
```

To get instance of `IRdbms` instance, just get it from dependency container as shown in following code.


```
var db : IRdbms;
...
db := container.get('db') as IRdbms;
```
or with array-like syntax

```
var db : IRdbms;
...
db := container['db'] as IRdbms;
```

## Executing SQL Query

```
var
    db : IRdbms;
    resultSet : IModelResultSet;
...
resultSet := db.prepare('SELECT * FROM users').execute();
```

### Passing parameters to SQL query

To avoid SQL injection, it is recommended to use prepared statement with parameter

```
resultSet := db.prepare('SELECT * FROM users WHERE user_email = :userEmail')
    .paramStr('userEmail', 'john@fanoframework.github.io')
    .execute();
```

## Get total rows in result set

```
var totData : integer;
...
totData := resultSet.resultCount();
```

## Read data from result set

```
var userId : integer;
    userEmail : string;
...
userId := resultSet.fields().fieldByName('user_id').asInteger;
userEmail := resultSet.fields().fieldByName('user_email').asString;
```

## Advance cursor to next position

`fields()` will read data in current cursor position. To read next data in result set, it is required that you call `next()` to advance cursor position.

```
resultSet.next();
```

## Test if at end of result set

`fieldByName()` method will throws exception if you try to read data when cursor is at end of file. To avoid it you need to check end of file condition with `eof()`. So to read all data in result set, you need following loop.

```
while not resultSet.eof() do
begin
    userId := resultSet.fields().fieldByName('user_id').asInteger;
    //do something with userId
    resultSet.next();
end;
```

## Executing Transaction

```
db.beginTransaction();
try
    //do multiple database operations
finally
    db.commit();
end;
```

## What can go wrong

You may find thing does not work due to missing library for example you [do not have MySQL client library](/known-issues#missing-mysql-client-library) or [ODBC client library](/known-issues#missing-odbc-client-library) installed.

## Explore more

- [Working with Models](/working-with-models)
- [IRdbms interface](https://github.com/fanoframework/fano/blob/master/src/Db/Rdbms/Contracts/RdbmsIntf.pas).
- [TMySQLDb class](https://github.com/fanoframework/fano/blob/master/src/Db/Rdbms/Implementations/MySql/MySqlDbImpl.pas).
- [TPostgreSqlDb class](https://github.com/fanoframework/fano/blob/master/src/Db/Rdbms/Implementations/PostgreSql/PostgreSqlDbImpl.pas).
- [TFirebirdDb class](https://github.com/fanoframework/fano/blob/master/src/Db/Rdbms/Implementations/Firebird/FirebirdDbImpl.pas).
- [TOdbcDb ckass](https://github.com/fanoframework/fano/blob/master/src/Db/Rdbms/Implementations/Odbc/OdbcDbImpl.pas).
- [TSQLiteDb class](https://github.com/fanoframework/fano/blob/master/src/Db/Rdbms/Implementations/SQLite/SQLiteDbImpl.pas).