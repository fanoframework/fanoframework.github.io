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

Built-in `TView` class dependends on instance of `ITemplateParser` interface and
and content of HTML template. `ITemplateParser` instance will be responsible to
parse variable placeholder, for example `{{varName}}`, in template with actual value.

```
var fileReader : IFileReader;
    templateParser : ITemplateParser;    
    view : IView;
...
templateParser:= TSimpleTemplateParser.create('{{', '}}');
fileReader:= TStringFileReader.create();
view := TView.create(
    templateParser,
    fileReader.readFile(
        extractFileDir(getCurrentDir()) + '/src/Templates/header.html'
    )
);
```

If your HTML template does not contain any variable placeholders to be replaced with value, you can modify code above and replace `templateParser` with `TNullTemplateParser`.

```
templateParser:= TNullTemplateParser.create();
```

There is `TTemplateParser` class which does similar thing as `TSimpleTemplateParser`. `TTemplateParser` class utilizes regular expression
to replace variable placeholders and more flexible.

For example using `TTemplateParser` class, you can add whitespaces between open and closing tag, so `{{ "{{ varName " }}}}` and `{{ "{{varName" }}}}` are considered same variable. `TSimpleTemplateParser` class replaces variable placeholders as exact
string replacement and only support `{{ "{{varName" }}}}` format but it is faster because it does not use regex.


### Load template from single HTML file

## Compose view from several views

## Displaying dynamic data

### Working with view parameter

### Working with array of data

## Displaying data from model
