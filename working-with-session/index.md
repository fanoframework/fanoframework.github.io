---
title: Working with Session
description: Tutorial on how to work with session in Fano Framework
---

<h1 class="major">Working with Session</h1>

## What is session?

HTTP protocol is stateless protocol. It does not have built-in mechanism to maintain
state between each request.

Session is mechanism which store request state in server. State is identified by some
random unique id. Each request and response will exchange this identifier to maintain
state between requests.

## ISessionManager

To use session in Fano Framework, you need to use `ISessionManager` and also dispatcher instance which support session. Fano Framework provides built-in implementation for this interface.

- `TFileSessionManager`, session manager which store session data in file.
- `TCookieSessionManager`, session manager which store session data in encrypted cookie.
- `TDbSessionManager`, session manager which store session data in [RDBMS database](/database).

## IReadOnlySessionManager

This interface is provided for getting session instance from a request. This interface is parent of `ISessionManager` interface and only has one method `getSession()` which expects `IRequest` instance and returns `ISession` instance.

## Create session manager instance

Easiest steps to work with session is to [scaffold Fano Framework web application project with session support](/scaffolding-with-fano-cli#add-session-support) using Fano CLI.

### Store session data in file

To register `TFileSessionManager` instance to dependency container, Fano Framework provides `TJsonFileSessionManagerFactory` and `TIniFileSessionManagerFactory` classes which will create session manager which store its data as JSON and INI file respectively.

Its constructor accepts three optional parameters:

- Name of session (default value of `FANOSESSID`). This will be used as name of cookie.
- Path of directory where files will be stored (default value of `/tmp`).
- Prefix which will be prepended before session id (default value of empty string).

You need to make sure that session directory is writeable by application.

```
var sessionMgrFactory : IDependencyFactory;
...
sessionMgrFactory := TJsonFileSessionManagerFactory.create(
    'fano_sess`,
    '/home/fanoapp/storages/sessions/'
);
```
or you can create factory with default value
```
sessionMgrFactory := TJsonFileSessionManagerFactory.create();
```

If it is not set, by default, code above will use GUID as session id (using `TGuidSessionIdGeneratorFactory` class). Read [Session ID generator](/working-with-session#session-id-generator) for more information on algorithm used to generate session identifier.

After that register factory to dependency container

```
container.add('sessionManager', sessionMgrFactory);
```

### Store session data in cookie

Fano Framework can store session data as encrypted cookie instead. This has advantages:

- Reduce disk usage compared to store session data in file. Session data is stored in user client browser cookies.
- Simplify session management when using load balancer running multiple application instances.

However, it has drawback too:

- [A single cookie have limit of 4KB in size](https://tools.ietf.org/html/rfc6265#section-6.1). You cannot store too many data and the size of actual unencrypted data may be less due to usage of base64 encoding.

To store session data in encrypted cookie value, you need to use `TCookieSessionManager`. You also need to create instance of `IEncrypter` and `IDecrypter` interface which responsible to encrypt and decrypt cookie value. Fano Framework provides built-in encrypter using Blowfish algorithm.

Following code show how to create session manager which store session data in encrypted cookie. You need to setup a random secure secret key to be used to encrypt and decrypt.

```
container.add(
    'encrypter',
    TBlowfishEncrypterFactory.create()
        .secretKey(
            config.getString('secretKey')
        )
);

container.add(
    'sessionManager',
    TCookieSessionManagerFactory.create(
        TJsonSessionFactory.create(),
        container['encrypter'] as IEncrypter,
        container['encrypter'] as IDecrypter,
        config.getString('session.name')
    )
);
```
Code above will internally store data as JSON format, to use INI format, just replace `TJsonSessionFactory` with `TIniSessionFactory` class.

You can replace `TBlowfishEncrypterFactory` above with `TSha1BlowfishEncrypterFactory` or `TMd5BlowfishEncrypterFactory` which adds encrypted cookie integrity check using HMAC SHA1 or HMAC MD5 respectively. When encrypted cookie is tampered, integrity check will fail and in turn, new session is created.

```
container.add(
    'encrypter',
    TSha1BlowfishEncrypterFactory.create()
        .secretKey(
            config.getString('secretKey')
        )
);
```
Fourth parameter of constructor method of `TCookieSessionManagerFactory` is optional session name parameter with default value of `FANOSESSID`. So you can omit and use default value as follows.
```
container.add(
    'sessionManager',
    TCookieSessionManagerFactory.create(
        TJsonSessionFactory.create(),
        container['encrypter'] as IEncrypter,
        container['encrypter'] as IDecrypter
    )
);

```
See [Fano Session Cookie](https://github.com/fanoframework/fano-session-cookie), example web project to demonstrate how to use session that store its data in encrypted cookie.

### Store session data in database

To use database for session storage, you need to create a table which contain at least three columns which will store session id, serialized session data and session expiry. Two first columns should be string type column, such as `VARCHAR` and last column should be `DATETIME`. For best performance, session id column should be primary key or at least unique index.

You need to make sure that you create proper database credential which has `SELECT`, `INSERT`, `UPDATE` and `DELETE` privilege on table mentioned above.

`TDbSessionManager` class provides capability to manage session data in database.
To register it to dependency container, Fano Framework provides `TJsonDbSessionManagerFactory` and `TIniDbSessionManagerFactory` classes that will create session manager having capability to store its data in database as serialized JSON and INI string respectively.

Its constructor accepts two parameters, instance of `IRdbms` interface and name of session (optional with default value of `FANOSESSID`).

It provides several additional methods to let Fano Framework knows about your table schema. Read [database documentation](/database) for information on working with database in Fano Framework.

```
var sessionMgrFactory : IDependencyFactory;
...
sessionMgrFactory := TJsonDbSessionManagerFactory.create(
    container['db'] as IRdbms,
    'fano_sess`
).table('fano_sessions')
.sessionIdColumn('id')
.dataColumn('data')
.expiredAtColumn('expired_at');

container.add('sessionManager', sessionMgrFactory);
```
If table schema is not defined, factory class will assume table name `fano_sessions` with three columns: `id`, `data` and `expired_at`. So following code is doing same thing as above.
```
sessionMgrFactory := TJsonDbSessionManagerFactory.create(
    container['db'] as IRdbms,
    'fano_sess`
);
```
You can omit last parameter and use default value as follows.
```
sessionMgrFactory := TJsonDbSessionManagerFactory.create(
    container['db'] as IRdbms
);
```
## <a name="session-id-generator"></a>Session ID generator

Fano Framework allows developer to change how session identifiers are generated. The idea is to minimise the probability of generating two session IDs with the same value.

You can change the way session identifiers are generated by calling `sessionIdGenerator()` of session manager factory and pass factory class of `ISessionIdGeneratorFactory` interface, as shown in following code,

```
sessionMgrFactory := TJsonFileSessionManagerFactory.create(
    'fano_sess`,
    '/home/fanoapp/storages/sessions/'
).sessionIdGenerator(
    TKeyGuidSessionIdGeneratorFactory.create('some random string as secret key')
);
```
All session manager factory classes have `sessionIdGenerator()` method so you can also set session id generator on, for example, `TJsonDbSessionManagerFactory`,
```
sessionMgrFactory := TJsonDbSessionManagerFactory.create(
    container['db'] as IRdbms,
    'fano_sess`
).sessionIdGenerator(
    TKeyGuidSessionIdGeneratorFactory.create('some random string as secret key')
).table('fano_sessions')
.sessionIdColumn('sess_id')
.dataColumn('sess_data')
.expiredAtColumn('expired_at');
```

If you need to implement your own session id generator, you need to implement `ISessionIdGenerator` interface and also create its factory class which implements `ISessionIdGeneratorFactory` interface.

### Built-in session id generator

- `TGuidSessionIdGeneratorFactory` is built-in factory class which will create session id generator which use GUID. While GUID is unique, it is predictable so it is vulnerable to session hijack attack.
- `TKeyGuidSessionIdGeneratorFactory` is built-in factory class which will create session id which use SHA1 hash of a secret key concatenated with GUID as session id.
- `TIpKeyGuidSessionIdGeneratorFactory` is built-in factory class which will create session id which use SHA1 hash of concatenated string of client IP address + time +  secret key + GUID as session id.
- `TKeyRandSessionIdGeneratorFactory` is built-in factory class which will create session id generator which use SHA1 hash of a secret key + client IP address + time + random bytes from `/dev/urandom`.
- `TSha2KeyRandSessionIdGeneratorFactory` is similar to `TKeyRandSessionIdGeneratorFactory` except that it uses SHA2 256-bit hash.

Except `TGuidSessionIdGeneratorFactory` which its constructor does not require parameter, other built-in factory classes expect secret key to be provided when creating factory class.
It is strongly advised that you use cryptographically strong random secret key. Fano CLI can help [generate random secret key](/scaffolding-with-fano-cli#generate-random-key) for you.

```
sessionMgrFactory := TJsonFileSessionManagerFactory.create(
    'fano_sess`,
    '/home/fanoapp/storages/sessions/'
).sessionIdGenerator(
    TGuidSessionIdGeneratorFactory.create()
);
```

or with more cryptographically strong session id generator

```
sessionMgrFactory := TJsonFileSessionManagerFactory.create(
    'fano_sess`,
    '/home/fanoapp/storages/sessions/'
).sessionIdGenerator(
    TSha2KeyRandSessionIdGeneratorFactory.create('your very secret hush hush key')
);
```
## Session initialization and serialization
To be able to keep track of client request, session data needs to be initialized each time request is coming and it needs to be serialized and persisted in storage when response is about to get sent over wire.

There is two mechanism where session initialization and serialization can be done, i.e via dispatcher and middleware.

When using dispatcher for working with session, session is initialized after request object is created and serialized to storage before response is sent to client over the wire. So session is always applied globally through out application.

When using middleware for working with session, session is initialized when middleware, which is attached globally to application middleware list or per route, is executed. Using middleware when working with session is more flexible. For example you may want to apply session management only for certain routes and not other.

## Create dispatcher instance which support session

`TSessionDispatcherFactory` is factory which can create dispatcher which support session.
It expects 6 parameters:

- Instance of global middleware collection.
- Instance of router matcher.
- Request response factory
- Instance of `ISessionManager` interface
- Instance of `ICookieFactory` interface
- Integer value of cookie max age in seconds

```
container.add(
    GuidToString(IDispatcher),
    TSessionDispatcherFactory.create(
        container.get('appMiddlewares') as IMiddlewareLinkList,
        container.get(GuidToString(IRouteMatcher)) as IRouteMatcher,
        TRequestResponseFactory.create(),
        container.get('sessionManager') as ISessionManager,
        (TCookieFactory.create()).domain('your.app.example.com'),
        cookieMaxAge
    )
);
```
Please read [Dispatcher](/dispatcher) for more information or you may want to get information about [how to create Fano web application project with session using Fano CLI](/scaffolding-with-fano-cli#add-session-support).

## Session middleware
Since v1.10.0, Fano Framework provides session middleware `TSessionMiddleware` which can be use, as an alternative to using dispatcher when working with session. `TSessionMiddlewareFactory` class is factory class for this middleware.

```
container.add(
    'my.session.middleware',
    TSessionMiddlewareFactory.create(
        container['sessionManager'] as ISessionManager
    )
);
```

To apply session middleware globally, so that session is initialized and serialized through out application.

```
(container['appMiddlewares'] as IMiddlewareList)
    .add(container['my.session.middleware'] as IMiddleware);
```

It is similar to using dispatcher.

To apply session middleware so that session is only initialized and serialized when certain routes is accessed, attach middleware to route.

```
router.get('/', container['myHomeCtrl'] as IRequestHandler)
    .add(container['my.session.middleware'] as IMiddleware);
router.get('/account', container['myAccountCtrl'] as IRequestHandler)
    .add(container['my.session.middleware'] as IMiddleware);
router.get('/staticfile.pdf', container['myFileCtrl'] as IRequestHandler);

```

Code above cause session to be initialized and persisted when route '/' and '/account' is accessed but not when '/staticfile.pdf' route is accessed.

## Injecting session manager instance to controller or middleware

```
type

    TAuthOnlyMiddleware = class(TInjectableObject, IMiddleware)
    private
        fSesionManager : IReadOnlySessionManager;
        ...
    public
        constructor create(const session : IReadOnlySessionManager);
        ...
    end;
...

implementation

    constructor TAuthOnlyMiddleware.create(const session : IReadOnlySessionManager);
    begin
        inherited create();
        fSession := session;
    end;
    ...
end.
```

## Get session instance

From inside controller or middleware, you can get `ISession` instance from request using `getSession()` method of `ISessionManager` or `IReadOnlySessionManager` interface.

```
function TAuthOnlyMiddleware.handleRequest(
        const request : IRequest;
        const response : IResponse;
        const args : IRouteArgsReader;
        const next : IRequestHandler
) : IResponse;
var sess : ISession;
begin
    sess := fSession.getSession(request);
    //do something with session instance
end;
```

After you hold `ISession` instance, you can query session state using its method.

You can also use simplified syntax,

```
sess := fSession[request];
```
## Test if session variable is set

```
var signedIn : boolean;
...
signedIn := sess.has('userSignedIn');
```

## Read session variable value

```
var userName : string;
...
userName := sess.getVar('username');
```

`getVar()` raise `ESessionExpired` exception if session is expired.

You can also use simplified syntax

```
userName := sess['username'];
```

## Write session variable value

```
sess.setVar('username', 'john@doe.com');
```

`setVar()` raise `ESessionExpired` exception if session is expired.

You can also use simplified syntax

```
sess['username'] := 'john@doe.com';
```

## Delete session variable

```
sess.delete('username');
```

## Delete all session variables

```
sess.clear();
```

## Test if session is expired

To avoid `ESessionExpired` exception when reading a value, you can test if current session is expired or not.

```
var userName : string;
...
if (not sess.expired()) then
begin
    userName := sess.getVar('username');
end;
```

## Get expiration datetime

To get time when session is expired

```
var expiresDate : TDatetime;
...
expiresDate := sess.expiresAt();
```

## Get current session id

To get current session id

```
var sessionId : string;
...
sessionId := sess.id();
```

## Get session name

To get session name,

```
var sessionName : string;
...
sessionName := sess.name();
```

If you create session manager factory as example above, `sessionName` will contains `fano_sess` value.


## Explore more

- [Dispatcher](/dispatcher)
- [Example applications](/examples)
- [Session example applications](https://github.com/fanoframework/fano-session)
- [Session in cookie example applications](https://github.com/fanoframework/fano-session-cookie)
- [Session in database example](https://github.com/fanoframework/fano-db-session)
