---
title: Identifying Client User-Agent
description: Tutorial on how to detect client browser, device type and platform with user agent utilities provided by Fano Framework
---

<h1 class="major">Identifying Client User-Agent</h1>

## About User-Agent

Most standard browsers will send identification string with each request header so that application running on server can use it to identify client browser, platform and device type. This is useful, for example, if you want to serve different page layout if user is using mobile device.

User-agent identification string is not reliable as they can be changed rather easily so that identification may yield false result.

## IUserAgent interface

To abstract away user-agent identification functionality implementation, `IUserAgent` interface is introduced. It is a contract for any class having capability to handle user-agent identification.

This interface provides methods to set and retrieve user-agent string and to retrive instance of several utility interfaces.

### IClientDevice interface

Interface of any class having capability to identify client device such mobile device or desktop computer. To retrieve `IClientDevice` instance, use `getDevice()` method of `IUserAgent` interface.

### IClientBrowser interface

Interface of any class having capability to identify client browser such Chrome or Firefox. To retrieve `IClientBrowser` instance, use `getBrowser()` method of `IUserAgent` interface.

### IClientOS interface

Interface of any class having capability to identify client operating system such as Android or iOS. To retrieve `IClientOS` instance, use `getOS()` method of `IUserAgent` interface.

## Setting up IUserAgent instance

Current implementation of `IInterface` interface only supports one implementation through class `TUserAgent`. To use it, register its factory class with [container](/dependency-container).

```
container.factory('ua', TUserAgentFactory.create());
```
to retrieve `IUserAgent` instance,
```
var auserAgent : IUserAgent;
...
auserAgent := container['ua'] as IUserAgent;
```

## Set user-agent string

Before you can identify client user-agent, you need to set user-agent string to instance of `IUserAgent`. User-agent of each request can be read from `IRequest` instance.

```
auserAgent.userAgent := request.headers().getHeader('User-Agent');
```
or
```
auserAgent.userAgent := request.env.httpUserAgent();
```

## Detecting if client is using mobile device

Use `isMobile()` method of `IClientDevice` interface to test if client is using mobile device.

```
var dev : IClientDevice;
...
dev := auserAgent.getDevice();
if dev.isMobile() then
begin
    //client using mobile device
end;
```

## Detecting client browser

Use `isBrowser()` method or `browser` property of `IClientBrowser` interface to test client browser.

```
var browserInst : IClientBrowser;
...
browserInst := auserAgent.getBrowser();
if browserInst.browser['Chrome'] then
begin
    //client using Chrome browser
end;

if browserInst.browser['Firefox'] then
begin
    //client using Firefox browser
end;
```

## Detecting client operating system

Use `isOS()` method or `OS` property of `IClientOS` interface to test client operating system.

```
var osInst : IClientOS;
...
osInst := auserAgent.getOS();
if osInst.OS['AndroidOS'] then
begin
    //client using Android device
end;

if osInst.OS['iOS'] then
begin
    //client using iOS device
end;
```

## Explore more

- [Utilities](/utilities)
- [Fano User-Agent](https://github.com/fanoframework/fano-user-agent) example project demonstrates how to work with user-agent.
