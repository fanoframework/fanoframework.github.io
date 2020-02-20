---
title: Displaying Data in View
description: Tutorial on how to display data in view
---

<h1 class="major">Displaying Data in View</h1>

To display dynamic data into `TView`, you will need to pass `ITemplateParser` interface instance which provide functionality to replace variable placeholder.

## Working with view parameter

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

Fano Framework provides several `IViewParameters` implementations.

- `TViewParameters`, basic implementation which can set and get view parameters.
- `TCompositeViewParameters`, class which composed from two external `IViewParameters` instances. This is mostly used to combine two view parameters as one.
- `TNullViewParameters`, null class implements `IViewParameters` instances.

## Working with array of data

Built-in implementation of template parser is basically just string manipulation operation and does not offer sophisticated template engine.

This should not be a problem when you need to display, for example, a list of
items, because you can always create your own `IView` interface implementation
to display that data.

For example if we have array of `TUserData` which declared as follows:

```
type

    TUserData = record
        id : integer;
        name : string;
        email : string;
    end;
    TUserDataArr = array of TUserData;
...
var userData : TUserDataArr;
```

Then you can create `IView` implementation that dis

```
(*!------------------------------------------------
 * render view
 *------------------------------------------------
 * @param viewParams view parameters
 * @param response response instance
 * @return response
 *-----------------------------------------------*)
function TUserListingView.render(
    const viewParams : IViewParameters;
    const response : IResponse
) : IResponse;
var respBody : IResponseStream;
    i : integer;
begin
    if (length(userData) > 0) then
    begin
        respBody.write(
            '<table>' +
            '<thead>' +
              '<tr>' +
              '  <th>No</th>' +
              '  <th>Name</th>' +
              '  <th>Email</th>' +
              '</tr>' +
            '</thead><tbody>'
        );
        for i:=0 to length(userData)-1 do
        begin
            respBody.write(
                '<tr>' +
                '<td>' + intToStr(userData[i].id) + '</td>' +
                '<td>' + userData[i].name' + '</td>' +
                '<td>' + userData[i].email + '</td>' +
                '</tr>'
            );
        end;
        respBody.write('</tbody></table>');
    end;
    result := response;
end;
```

## Displaying data from model

[Working with Models](/working-with-models) explains about model that Fano Framework use.

To work with models in view instance, what you need to do is to inject model
instance into view

Following snippet code is view implementation that display data from static JSON file. It is part of [fano-mvc](https://github.com/fanoframework/fano-mvc) sample application.

```
(*!------------------------------------------------
 * render view
 *------------------------------------------------
 * @param viewParams view parameters
 * @param response response instance
 * @return response
 *-----------------------------------------------*)
function TUserListingView.render(
    const viewParams : IViewParameters;
    const response : IResponse
) : IResponse;
var userData : IModelResultSet;
    respBody : IResponseStream;
begin
    userData := userModel.data();
    respBody := response.body();
    if (userData.count() > 0) then
    begin
        respBody.write(
            '<table>' +
            '<thead>' +
              '<tr>' +
              '  <th>No</th>' +
              '  <th>Name</th>' +
              '  <th>Email</th>' +
              '</tr>' +
            '</thead><tbody>'
        );
        while (not userData.eof()) do
        begin
            respBody.write(
                '<tr>' +
                '<td>' + userData.readString('id') + '</td>' +
                '<td>' + userData.readString('name') + '</td>' +
                '<td>' + userData.readString('email') + '</td>' +
                '</tr>'
            );
            userData.next();
        end;
        respBody.write('</tbody></table>');
    end;
    result := response;
end;
```

## Explore more

- [Working with Views](/working-with-views)
- [Working with Controllers](/working-with-controllers)
- [Working with Models](/working-with-models)
