---
title: Form Validation
description: Handling form validation in Fano Framework
---

<h1 class="major">Form Validation</h1>

We should never trust data coming from user input. Fano Framework provides mechanism
for web application developer to validate data.

## Working with IRequestValidator and IValidationRules

`IRequestValidator` is interface for any class having capability to validate request and returns validation result.

While `IValidationRules` is interface for any class having capability to manage one or more validation rule.

Fano Framework provides `TValidation` class which implements both interfaces.

```
var requestValidator : IRequestValidator;
    validatorRules : IValidationRules;
...
requestValidator := TValidation.create(THashList.create());
validatorRules := requestValidator as IValidationRules;
```

## Validation rule

A validation rule is class which responsible to decide if data being processed is valid or not. Any class implements `IValidator` interface can be used as validation rule and it must implements following methods,

```
function isValid(
    const key : shortstring;
    const dataToValidate : IList;
    const request : IRequest
) : boolean;

function errorMessage(const key : shortstring) : string;
```

### isValid() method

This method expects following parameters,

- `key`, name of field to be validated,
- `dataToValidate`, instance which contains query strings or request body parameters,
- `request`, instance of current request

It must return boolean value of `true` if validation should pass or `false` otherwise.

### errorMessage() method

This method expects following parameters,

- `key`, name of field to be validated,

It must return string of validation error message.

Developer do not call both methods directly. It will be called by `IRequestValidator` instance when validating request.

## Add validation rules

`IValidationRules` interface has methods `addRule()` which developer can use to add a validation rule and associate it with field. First parameter is field name and second is validation rule.

For example if you have following form,

```
<form id="frmCreateUser" method="post" action="/create-user">
    <label for="usernameInput">Username</label>
    <input id="usernameInput" name="username" type="text">
    <label for="emailInput">Email</label>
    <input id="emailInput" name="email" type="text">
</form>
```

You need that both username and email are mandatory. Username is restricted to alpha numeric and dash character only. Email must conform to email address format.

Following validation rules are examples to satisfy above requirement.

```
validationRules
.addRule(
    'username',
    TCompositeValidator.create([
        TRequiredValidator.create(),
        TAlphaNumDashValidator.create(TRegex.create())
    ])
)
.addRule(
    'email',
    TCompositeValidator.create([
        TRequiredValidator.create(),
        TEmailValidator.create(TRegex.create())
    ])
);
```
`TCompositeValidator` is built-in validation rule implementation which combine one or more validators as one so that complex validation rule can be achieved by combining several simple validation rules.

## Validate request

You can either do validation in middleware or in controller. If same validation rules can be applied to several controllers, then validation in middleware is more suitable.

To validate a request, you need to call `validate()` method of `IRequestValidator` instance and pass current request instance. It returns `TValidationResult` record type which is declared as follows

```
type
    TValidationMessage = record
        key : shortstring;
        errorMessage : string;
    end;
    TValidationMessages = array of TValidationMessage;

    TValidationResult = record
        isValid : boolean;
        errorMessages : TValidationMessages;
    end;
```

For example, here we validate request inside controller

```
function TMyController.handleRequest(
    const request : IRequest;
    const response : IResponse;
    const args : IRouteArgsReader
) : IResponse;
var i, len : integer;
    validationRes : TValidationResult;
begin
    if fValidation.validate(request).isValid then
    begin
        //validation success
        response.body().write('Hey things are OK');
    end else
    begin
        validationRes := fValidation.lastValidationResult();
        response.body().write('Validation fails:');
        response.body().write('<ul>');
        len := length(validationRes.errorMessages);
        for i := 0 to len - 1 do
        begin
            response.body().write(
               '<li>' + validationRes.errorMessages[i].errorMessage + '</li>'
            );
        end;
        response.body().write('</ul>');
    end;
    result := response;
end;
```

Please note that `lastValidationResult()` return last validation status so, above example can be simplified by removing `lastValidationResult()` call as follows

```
    validationRes := fValidation.validate(request);
    if validationRes.isValid then
    begin
        //validation success
        response.body().write('Hey things are OK');
    end else
    begin
        response.body().write('Validation fails:');
        response.body().write('<ul>');
        len := length(validationRes.errorMessages);
        ...
    end;
    ...
```

`lastValidationResult()` method is provided so developer can execute validation in middleware and later in controller inspects last validation status and act accordingly.

## Built-in validation rule

Fano Framework comes with several built-in validation rules, some are validation rule mentioned in above code example. Read [Built-in Validation Rules](/security/form-validation/built-in-validation-rules) for more information.

## Writing your own validation rule

If built-in validation rules do not meet your requirement, you can create your own validation rule by creating class that implements `IValidator`.

To simplify, Fano Framework provides `TBaseValidator` class which developer can use to create validation rule. It is abstract class which developer required to implements its protected method `isValidData()`

```
function isValidData(
    const dataToValidate : string;
    const dataCollection : IReadOnlyList;
    const request : IRequest
) : boolean; virtual; abstract;
```

- `dataToValidate` will contain actual data of current field being validated.
- `dataCollection` will contain list of key value pair of all query strings and POST body of request.
- `request` will contain current request object, in case you need to more informations.

Your implementation must return `true` if validation should pass or `false` otherwise.

For example, following code is implementation of custom validation rule which pass validation only if data being validated is registered and active user id.

```
unit ActiveUserValidatorImpl;

interface

{$MODE OBJFPC}
{$H+}

uses

    fano;

type

    TActiveUserValidator = class(TBaseValidator)
    private
        fDatabase : IRdbms;
    protected
        function isValidData(
            const userId : string;
            const dataCollection : IList;
            const request : IRequest
        ) : boolean; override;
    public
        constructor create(const db : IRdbms);
        destructor destroy(); override
    end;

implementation

resourcestring

    sErrFieldMustBeActiveUserId = 'Field %s must be active user id';

    constructor TActiveUserValidator.create(const db : IRdbms);
    begin
        inherited create(sErrFieldMustBeActiveUserId);
        fDatabase := db;
    end;

    destructor TActiveUserValidator.destroy();
    begin
        fDatabase := nil;
        inherited destroy();
    end;

    function TActiveUserValidator.isValidData(
        const userId : string;
        const dataCollection : IList;
        const request : IRequest
    ) : boolean;
    var sql : string;
    begin
        sql := 'SELECT user_id FROM users '+
            'WHERE user_id = :userId AND status = :userStatus LIMIT 1';
        result := (fDatabase
            .prepare(sql)
            .paramStr('userId', userId)
            .paramStr('userStatus', 'active')
            .execute()
            .resultCount() > 0);
    end;
end.

```

## Explore more

- [Built-in Validation Rules](/security/form-validation/built-in-validation-rules)
- [Working with request](/working-with-request)
- [Middlewares](/middlewares)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
