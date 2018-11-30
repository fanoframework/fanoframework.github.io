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

Built-in `TView` class depends on instance of `ITemplateParser` interface and
and content of HTML template. `ITemplateParser` instance will be responsible to
parse variable placeholder, for example `[[varName]]`, in template with actual value.

```
uses fano;
...
var templateParser : ITemplateParser;    
    view : IView;
...
templateParser:= TSimpleTemplateParser.create('[[', ']]');
view := TView.create(
    templateParser,
    '<html><head><title>Hello</title></head>' +
    '<body>Hello [[varName]]</body></html>'
);
```

If your HTML template does not contain any variable placeholders to be replaced with value, you can modify code above and replace `templateParser` with `TNullTemplateParser`.

```
templateParser:= TNullTemplateParser.create();
```

There is `TTemplateParser` class which does similar thing as `TSimpleTemplateParser`. `TTemplateParser` class utilizes regular expression
to replace variable placeholders and more flexible.

For example using `TTemplateParser` class, you can add whitespaces between open and closing tag, so `[[ varName ]]` and `[[varName]]` are considered same variable.

`TSimpleTemplateParser` class replaces variable placeholders with exact
string replacement and only support `[[varName]]` format but it is faster because it does not use regular expression.


### Load template from single HTML file

Fano comes with built-in utility to read file content and store it in
string with `IFileReader` interface. This interface has one method `readFile()` as follows:

```
(*!------------------------------------------------
 * read file content to string
 *-----------------------------------------------
 * @param filePath path of file to be read
 * @return file content
 *-----------------------------------------------*)
function readFile(const filePath : string) : string;
```

To initialize `IView` from HTML template file, you can use following code snippet.

```
uses fano;
...
var fileReader : IFileReader;
    templateParser : ITemplateParser;    
    view : IView;
...
templateParser:= TSimpleTemplateParser.create('[[', ']]');
fileReader:= TStringFileReader.create();
view := TView.create(
    templateParser,
    fileReader.readFile(
        '/path/to/templates/file.html'
    )
);
```

## Compose view from several views

Fano has `TCompositeView`, built-in class that implements `IView` interface which render view from two `IView` interface instances.

For example, if you have following HTML template structure

```
<!-- start of header part -->
<html>
<head>
</head>
<body>
<!-- end of header part -->

<!-- main content -->    

<!-- start footer part -->
</body>
</html>
<!-- end footer part -->
```

where header part and footer part mostly similar between pages.

You can create `IView` instances that render header, main content and footer
and then compose them as one view with `TCompositeView`.

### Header part

```
uses fano;
...
var headerView : IView;
...
headerView := TView.create(
    templateParser,
    '<!-- start of header part -->' +
    '<html><head></head><body>' +
    '<!-- end of header part -->'
);
```

### Footer part

```
uses fano;
...
var footerView : IView;
...
footerView := TView.create(
    templateParser,
    '<!-- start footer part -->' +
    '</body></html>' +
    '<!-- end footer part -->'
);
```

### Main content part

```
uses fano;
...
var contentView : IView;
...
contentView := TView.create(
    templateParser,
    '<!-- main content part -->'
);
```

### Composed view

`TCompositeView` supports composing two views. To compose more than two views you need to daisy-chained them, for example:

```
uses fano;
...
var view : IView;
...
view := TCompositeView.create(
    TCompositeView.create(headerView, contentView)
    footerView
);
```

## Displaying dynamic data

To display dynamic data into `TView`, you will need to pass `ITemplateParser` interface instance which provide functionality to replace variable placeholder.

### Working with view parameter

`render()` method from `IView` interface accepts instance `IViewParameters` as its first parameter. Controller will be responsible to provide view parameters.

`IViewParameters` interface has three methods:

- `getVar()`, method that return string value of variable name
- `setVar()`, method that set string value of variable name
- `vars()`, returns all variable name as TStrings

If you set variable name such as following code:

```
uses fano;
...
var viewParams : IViewParameters;
...
viewParams.setVar('foo', 'Hello').setVar('bar', 'World');
```

And you have following HTML template

```
<html>
<head><!--[foo]--></head>
<body>
<div class="foo"><!--[foo]--><div>
<div class="bar"><!--[bar]--><div>
</body>
</html>
```

with template parser instance defined as follow

```
templateParser:= TSimpleTemplateParser.create('<!--[', ']-->');
```

Output response will be

```
<html>
<head>Hello</head>
<body>
<div class="foo">Hello<div>
<div class="bar">World<div>
</body>
</html>
```

### Working with array of data

Built-in implementation of template parser is basically just string manipulation operation and does not offer sophisticated template engine.

This should not be a problem when you need to display, for example, a list of
items.

## Displaying data from model
