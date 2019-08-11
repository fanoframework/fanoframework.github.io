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
- `response` is class instance that implements `IResponse` interface. Read [Working with Response](/working-with-response) for information about `IResponse`.

`render()` method expects that you will return response instance that will be used to render HTTP response.

## Working with template

Fano provides some built-in classes implementing `IView` interface which you can use to work with HTML template to display web application response.

- `TView`, class that display view from HTML template string.
- `TTemplateView`, class that display view from HTML template file.
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

While `TView` can load HTML from file, `TTemplateView` is provided to simplify little bit,

```
uses fano;
...
view := TTemplateView.create(
    '/path/to/templates/file.html'
    templateParser,
    fileReader
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

## Compose view from one or more view partials

View partial is basically any class which implements `IViewPartial` interface. This interface has only one method `partial()` that implementor must provide.
Given template path and view parameters, `partial()` method implementation must returns view content as string with any variable placeholders replaced with actual values.

```
(*!------------------------------------------------
 * read template file, parse and replace its variable
 * and output to string
 *-----------------------------------------------
 * @param templatePath file path of template
 * @param viewParams instance contains view parameters
 * @return string which all variables replaced with value from
 *        view parameters
 *-----------------------------------------------*)
function partial(
    const templatePath : string;
    const viewParams : IViewParameters
) : string;
```

See *Working with view parameter* section in [Displaying Data in View](/working-with-views/displaying-data-in-view) for information how to work with `IViewParameters`.

For example, if you have following HTML template structure

`/path/to/templates/main.template.html`

```
<html>
<head>
    <!--[topCssJs]-->
</head>
<body>
<!--[navbar]-->
<!--[content]-->
<!--[footerJs]-->
</body>
</html>
```

`/path/to/templates/top.css.js.html`

```
<title>Hello</title>
```

`/path/to/templates/navbar.html`

```
<ul>
<li>Home</li>
</ul>
```

`/path/to/templates/footer.html`

```
<div>This is Footer</div>
```

You can create main view class,

```
(*!------------------------------------------------
 * render view
 *------------------------------------------------
 * @param viewParams view parameters
 * @param response response instance
 * @return response
*-----------------------------------------------*)
function TMainView.render(
    const viewParams : IViewParameters;
    const response : IResponse
) : IResponse;
begin
    viewParams.setVar(
        'topCssJs',
        fViewPartial.partial(
            '/path/to/templates/top.css.js.html',
            viewParams
        )
    );

    viewParams.setVar(
        'navbar',
        fViewPartial.partial(
            '/path/to/templates/navbar.html',
            viewParams
        )
    );

    viewParams.setVar(
        'footerJs',
        fViewPartial.partial(
            '/path/to/templates/footer.js.html',
            viewParams
        )
    );

    response.body().write(fTemplateParser.parse(fTemplateStr, viewParams));
    result := response;
end;

```

and create main view instance as follows,

```
var mainView : IView;

mainView := TMainView.create(
    '/path/to/templates/main.template.html'
    templateParser,
    fileReader
);

```

When main view is rendered by `TMyController` inherited from base class [`TController`](https://github.com/fanoframework/fano/blob/master/src/Mvc/Controllers/ControllerImpl.pas), as follows

```
type

    TMyController = class(TController)
    public
        function handleRequest(
            const request : IRequest;
            const response : IResponse
        ) : IResponse; override;
    end;

...

function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse
) : IResponse;
begin
    viewParams.setVar('content', '<p>Hello World</p>');
    inherited handleRequest(request, response);
end;
```

It will output HTML response as follows

```
<html>
<head>
    <title>Hello</title>
</head>
<body>
<ul>
<li>Home</li>
</ul>
<p>Hello World</p>
<div>This is Footer</div>
</body>
</html>
```

## Explore more

- [Displaying Data in View](/working-with-views/displaying-data-in-view)
- [Working with Controllers](/working-with-controllers)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
