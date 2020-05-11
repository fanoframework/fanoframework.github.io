---
title: Working with Database
description: Working with database in Fano Framework
---

<h1 class="major">Working with Database</h1>

## IRdbms interface

IRdbms is just a thin wrapper for Free Pascal SQLdb package. Currently supported RDBMS system is

- MySQL via `TMysqlDb` class
- PostgreSQL via `TPostgreSqlDb` class
- Firebird via `TFirebirdDb` class,
- SQLite via `TSQLiteDb` class

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

## Executing SQL Query

```
var resultSet : IModelResultSet;
...
resultSet := db.prepare('SELECT * FROM users').execute();
```

### Passing parameters to SQL query

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

## Explore more

- [Working with Models](/working-with-models)
