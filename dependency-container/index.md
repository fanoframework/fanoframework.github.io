---
title: Dependency Container
description: Tutorial on how to work with dependency container in Fano Framework
---

<h1 class="major">Dependency Container</h1>

## Why dependency container?

Fano Framework is designed to be extensible. It uses class composition a lot.
Instead of creating class with deep inheritance, many classes in this framework which do complex thing depend on one or more classes that do simple thing.

Because of this, creating instance of a class may require us to create
other classes instance that it requires. Some classes may shared same dependency to same class, some classes may not. Some classes need same instance of
other class instance, while some other may requires new instance.

This task grows in complexity when scope of problem grows. So we need to provide a way to manage dependencies between software components in application.

## Container, factory and its service

Dependency container, or simply container, is software component that manages
any software components (a.k.a, service). In Fano Framework, dependency container
implementation must implements `IDependencyContainer` interface.

The services are registered into dependency container during service registration which later can be queried to retrieve service instance.

During service registration, a service name is associated with a factory class
that must implements `IDependencyFactory` interface that will responsible to create the required service.

For a service to be able to work with `IDependencyContainer` implementation, it must implements `IDependency` interface. Fano Framework comes with base class `TInjectableObject` that implements `IDependency` interface.

## Service registration

To register a service, dependency container provides two method `add()` and `factory()`. See "Single vs multiple instance"  section for explanation.

For example, following code registers service factory `TSimpleRouterFactory` with name `router`. This factory class will create `TRouter` class.

```
container.add('router', TSimpleRouterFactory.create());
```

## Retrieve service instance from dependency container

Later, to get instance of `router` from container,

```
var inst : IDependency;
...
inst := container.get('router');
```

In Fano Framework, `get()` method of `IDependencyContainer` always returns `IDependency` interface instance. So you need to convert it to its correct type
before you can use it.

```
var router : IRouteMatcher;
...
router := inst as IRouteMatcher;
```
or simply,

```
var router : IRouteMatcher;
...
router := container.get('router') as IRouteMatcher;
```

When `get()` can not find service, it raises `EDependencyNotFound` exception.
If service is registered but with nil factory class, `EInvalidFactory` exception is raised.

## Test if service is registered

Dependency container provides `has()` method which return boolean value that can be used to check if particular service is registered or not.

```
if (container.has('router')) then
begin
   //router is registered, do something
end;
```

## Single vs multiple instance

When a service is registered using `add()` method, it is registered as single instance service. So every time a service is queried, container returns
same instance.

In following example `router1` and `router2` will point to same instance.

```
container.add('router', TSimpleRouteCollectionFactory.create());
...
var router1, router2 : IRouteMatcher;
...
router1 := container.get('router') as IRouteMatcher;
router2 := container.get('router') as IRouteMatcher;
```

When a service is registered using `factory()` method, it is registered as multiple instance services. So every time a service is queried, container returns
different instance. In code below, `router1` and `router2` will point to different instance.

```
container.factory('router', TSimpleRouteCollectionFactory.create());
...
var router1, router2 : IRouteMatcher;
...
router1 := container.get('router') as IRouteMatcher;
router2 := container.get('router') as IRouteMatcher;
```

## Built-in dependency container

Fano Framework comes with built-in dependency container, `TDependencyContainer`.
Of course, you are free to implements your own dependency container, as long as it implements `IDependencyContainer` interface.

## Auto-wire dependency

Auto-wire dependency is not yet possible due Free Pascal's lack of annotation
feature.
