---
title: Dependency Container
description: Tutorial on how to work with dependency container in Fano Framework
---

<h1 class="major">Dependency Container</h1>

## Why dependency container?

Fano Framework is designed to be extensible. It uses class composition heavily.
Instead of creating class with deep inheritance, many complex classes in this framework depend on one or more classes that do simple thing.

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
that must implements `IDependencyFactory` interface that will be responsible to create the required service.

For a service to be able to work with `IDependencyContainer` implementation, it must implements `IDependency` interface.
`IDependency` does not declare any methods. So following declaration is suffice.

```
TMyClass = class(TInterfacedObject, IDependency);
```

Fano Framework comes with base class `TInjectableObject` that implements `IDependency` interface.

## Service registration

To register a service, dependency container provides two method `add()` and `factory()`. Both methods expects two parameters, shortstring value of service name and instance of
`IDependencyFactory` responsible for creating service. You can use any value for service identifier string. If same identifier is already registered, it will overwrite previous `IDependencyFactory`.
See [Single vs multiple instance](#single-vs-multiple-instance) section for explanation of both methods.

For example, following code registers service factory `TSimpleRouterFactory` with name `router`. This factory class will create `TRouter` class.

```
var container : IDependencyContainer;
...
container.add('router', TSimpleRouterFactory.create());
```

You can also use alternative way

```
container.add(GUIDToString(IRouteMatcher), TSimpleRouterFactory.create());
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

or if use GUID string as service name

```
var router : IRouteMatcher;
...
router := container.get(GUIDToString(IRouteMatcher)) as IRouteMatcher;
```

When `get()` can not find service, it raises `EDependencyNotFound` exception.
If service is registered but with nil factory class, `EInvalidFactory` exception is raised.

You can also use simplified array-like syntax, for example

```
router := container['router'] as IRouteMatcher;
```
or

```
router := container.services['router'] as IRouteMatcher;
```

## Test if service is registered

Dependency container provides `has()` method which return boolean value that can be used to check if particular service is registered or not.

```
if (container.has('router')) then
begin
   //router is registered, do something
end;
```

## <a name="single-vs-multiple-instance"></a>Single vs multiple instance

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

## Adding service alias

To register new name for an existing service, dependency container provides method `alias()`.
This method expects two parameters. First parameter is new name for existing service, and second parameter identifies existing service to be aliased.

For example, if you have service registered as follows,

```
container.add(GUIDToString(IRouteMatcher), TSimpleRouterFactory.create());
```

You can create alias for above service as follows,

```
container.alias(GUIDToString(IRouter), GUIDToString(IRouteMatcher));
```

Then you can retrieve instance using

```
var
   router : IRouter;
   routeMatcher : IRouteMatcher;
...
router := container.get(GUIDToString(IRouter)) as IRouter;
routeMatcher := container.get(GUIDToString(IRouteMatcher)) as IRouteMatcher;
```

Both `router` and `routeMatcher` will point to same class instance.


However, if you have service registered as multipe instances,

```
container.factory(GUIDToString(IRouteMatcher), TSimpleRouterFactory.create());
```

`router` and `routeMatcher` will point to different instance.


## Built-in dependency container

Fano Framework comes with built-in dependency container,

- `TDependencyContainer`, this is basic dependency container which store service registration in hash list.

Of course, you are free to implements your own dependency container, as long as it implements `IDependencyContainer` interface.

## Circular dependency issue

Circular dependency issue arises when a service depends on itself, directly or indirectly. For example, A needs B, B needs C, C needs A.

This causes unterminated recursion. Fano Framework will raise `ECircularDependency` exception to prevent such condition.

Following example shows circular dependency condition, which will trigger `ECircularDepedency` exception.

```
function THomeCtrlFactory.build(const cntr : IDependencyContainer) : IDependency;
begin
      result := cntr['homeCtrl'];
end;
...
container.add('homeCtrl', THomeCtrlFactory.create());
...
router.get('/', container['homeCtrl'] as IRequestHandler);
```

If you are in circular dependency situation, you need to rethink about your application architecture.

## Auto-wire dependency

Auto-wire dependency is not yet implemented.

## Explore more

- [Working with Application](/working-with-application)
