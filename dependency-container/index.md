---
title: Dependency Container
description: Tutorial on how to work with dependency container in Fano Framework
---

<h1 class="major">Dependency Container</h1>

## Why dependency container?

Fano Framework is designed to be extensible. It uses class composition a lot.
So instead of creating class with deep inheritance, class that does complex thing depends on one or more classes that does simple thing.

Because of this, creating instance of class may require us to create
a bunch of other classes instance that it required. Some classes may shared same dependency to same class, some classes may not. Some classes need same instance of
other class instance, while some other may requires new instance.

This task grows in complexity when scope of problem grows. So we need to provide a way to manage dependencies between software components in application.

## Container, factory and its service


## Built-in dependency container

## Auto-wire dependency
