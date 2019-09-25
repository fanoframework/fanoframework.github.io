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

## Create session manager instance

### Create factory class

To register `TFileSessionManager` instance to dependency container, Fano Framework provides `TJsonFileSessionManagerFactory` and `TIniFileSessionManagerFactory` classes which will create session manager which store its data as JSON and INI file respectively.

Its constructor accepts two parameters, name of session and path of directory where files will be stored. You need to make sure that directory writeable by application.

```
var sessionMgrFactory : IDependencyFactory;
...
sessionMgrFactory := TJsonFileSessionManagerFactory.create(
    'fano_sess`,
    '/home/fanoapp/storages/sessions/'
);
```

### Register factory to dependency container

```
container.add('sessionManager', sessionMgrFactory);
```

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

## Injecting session manager instance to controller or middleware

```
type

    TAuthOnlyMiddleware = class(TInjectableObject, IMiddleware)
    private
        fSesionManager : ISessionManager;
        ...
    public
        constructor create(const session : ISessionManager);
        ...
    end;
...

implementation

    constructor TAuthOnlyMiddleware.create(const session : ISessionManager);
    begin
        inherited create();
        fSession := session;
    end;
    ...
end.
```

## Get session instance

From inside controller or middleware, you can get `ISession` instance from request using `getSession()` method of `ISessionManager` interface.

```
function TAuthOnlyMiddleware.handleRequest(
        const request : IRequest;
        const response : IResponse;
        var canContinue : boolean
) : IResponse;
var sess : ISession;
begin
    sess := fSession.getSession(request);
    //do something with session instance
end;
```

After you hold `ISession` instance, you can query session state using its method.

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

## Write session variable value

```
sess.setVar('username', 'john@doe.com');
```

`setVar()` raise `ESessionExpired` exception if session is expired.

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

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
