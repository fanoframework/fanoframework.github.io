---
title: Working with Views
description: Tutorial on how to work with views in Fano Framework
---

<h1 class="major">Working with Views</h1>

## IView interface

Interface `IView`, declared in unit `fano.pas`, is basis of view implementation in Fano Framework. It consists of `render()` method that implementor class must provide.

```
(*!------------------------------------------------
 * render view
 *------------------------------------------------
 * @param viewParams view parameters
 * @param response response instance
 * @return response
 *-----------------------------------------------*)
function render(
    const viewParams : IViewParameters;
    const response : IResponse
) : IResponse;

```

There are two input parameters:

- `viewParams` is class instance that implements `IViewParameters` interface. It
store any variable replacement. See *Working with view parameter* section for more information.
- `response` is class instance that implements `IResponse` interface.

`render()` method expects that you will return response instance that will be used to render HTTP response.

## Working with template

Fano provides some built-in classes implementing `IView` interface which you can use to work with HTML template to display web application response.

- `TView`, class that display view from HTML template string.
- `TCompositeView`, class that display view from two other `IView` instance. This is provided so we can compose several views as one. For example display
template consist of header, main content and footer.
- `TNullView` class implements `IView` but does nothing. This class is provide
mostly in conjunction with TCompositeView

### Load template from string

### Load template from single HTML file

## Compose view from several views

## Displaying dynamic data

### Working with view parameter

### Working with array of data

## Displaying data from model
