---
title: Middlewares
description: Documentation about how middlewares work in Fano Framework
---

<h1 class="major">Middlewares</h1>

## Why middleware?

In Fano Framework, middleware is an optional software component that is executed before or after request is pass to actual request handler. It is similar concept to firewall in which it can pass, block or modify request.

For example, middleware allows developer to test if user is logged in before
request reaches controller, It not then it blocks request.
So when controller is executed, developer can be sure that user must be logged in.

Single middleware instance can be attach to one or more route. This allows
centralized action to be executed to multiple controllers.

## Middleware architecture in Fano Framework

## Creating middleware

## Attaching middleware to route
