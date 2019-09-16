---
title: Built-in validation rules
description: List of built-in validation rules in Fano Framework
---

<h1 class="major">Built-in Validation Rules</h1>

## Field  availability

Following validator test if field is present.

### TRequiredValidator

Field must be present and not empty.

```
var rule : IValidator;
...
rule := TRequiredValidator.create();
```

### TPresentValidator

Field must be present but allowed to be empty.

```
rule := TPresentValidator.create();
```

## Character format

### TAlphaValidator

Data must be alphabet character only. This class expect instance of `IRegex` interface.

```
rule := TAlphaValidator.create(TRegex.create());
```

### TAlphaNumValidator

Data must be alphabet and numeric character only.

```
rule := TAlphaNumValidator.create(TRegex.create());
```

### TAlphaNumSpaceValidator

Data must be alphabet, numeric and space character only.

```
rule := TAlphaNumSpaceValidator.create(TRegex.create());
```

### TAlphaNumDashValidator

Data must be alphabet, numeric and dash character only.

```
rule := TAlphaNumDashValidator.create(TRegex.create());
```

### TEmailValidator

Data must be in valid email format. This validator does not check if email address actually exists.

```
rule := TEmailValidator.create(TRegex.create());
```

### TRegexValidator

Data must be in format as specified by regular expression pattern.

```
rule := TRegexValidator.create(
    TRegex.create(),
    '[a-zA-Z]+',
    'Field %s does not match regular expression'
);
```

## String length

### TEqualLengthValidator

Data must be string with its length is equal to predefined value. For example, data being validated must contains exacly 10 characters to pass validation.

```
rule := TEqualLengthValidator.create(10);
```

### TMinLengthValidator

Data must be string with its length not less than predefined value.

```
rule := TMinLengthValidator.create(10);
```

### TMaxLengthValidator

Data must be string with its length not greter than predefined value.

```
rule := TMaxLengthValidator.create(10);
```

## Numeric data

### TIntegerValidator

Data must be integer value.

```
rule := TIntegerValidator.create();
```

### TFloatValidator

Data must be float value.

```
rule := TFloatValidator.create();
```

### TCurrencyValidator

Data must be currency value.

```
rule := TCurrencyValidator.create();
```

### TNumericValidator

Data must be integer or float value.

```
rule := TNumericValidator.create();
```

### TMinIntegerValidator

Data must be integer value and its value not less than predefined value.

```
rule := TMinIntegerValidator.create(100);
```
Validation will pass only if you pass integer value >= 100.

### TMaxIntegerValidator

Data must be integer value and not greater than predefined value.

```
rule := TMaxIntegerValidator.create(100);
```
Validation will pass only if you pass integer value <= 100.

## Boolean data

### TBooleanValidator

Data must be boolean value

```
rule := TBooleanValidator.create();
```

### TAcceptedValidator

Data must be boolean value `true`, or integer 1 or string `yes` or string `on`. This is mostly to be used for accepting "Terms and Conditions".

```
rule := TAcceptedValidator.create();
```

## Ranged data

### TInValidator

Data must be one of predefined values.

```
rule := TInValidator.create(['foo', 'bar']);
```
Validation will pass only if its value is equal to `foo` or `bar`.

### TNotInValidator

Data must not be one of predefined values.

```
rule := TNotInValidator.create(['foo', 'bar']);
```
Validation will pass only if its value is not equal to `foo` or `bar`.

## Date and time

### TDateValidator

Data must be valid date

```
rule := TDateValidator.create();
```

### TDateTimeValidator

Data must be valid date time.

```
rule := TDateTimeValidator.create();
```

### TTimeValidator

Data must be valid time.

```
rule := TTimeValidator.create();
```

### TEqualDateTimeValidator

Data must be date time and its value must equal predefined value.

```
rule := TEqualDateTimeValidator.create(date());
```

### TAfterDateTimeValidator

Data must be date time and its value must be later than predefined value.

```
rule := TAfterDateTimeValidator.create(date());
```

### TBeforeDateTimeValidator

Data must be date time and its value must be prior than predefined value.

```
rule := TBeforeDateTimeValidator.create(date());
```

## Uploaded file

### TUploadedFileValidator

Field must be valid uploaded file.

```
rule := TUploadedFileValidator.create();
```

### TUploadedMimeValidator

Field must be valid uploaded file and its content type must be one of predefined MIME types.

```
rule := TUploadedMimeValidator.create(['image/jpg', 'image/png']);
```

### TUploadedSizeValidator

Field must be valid uploaded file and its file size must be less or equal predefined size in bytes. For example to restrict that uploaded file size must not exceed 200KB.

```
rule := TUploadedSizeValidator.create(200 * 1024);
```

## Miscellaneous

### TCompositeValidator

Validator class which combined one or more validators as one. For example to make field to become mandatory and alpha numeric dash characters only, with length between 8 and 10,

```
rule := TCompositeValidator.create([
    TRequiredValidator.create(),
    TAlphaNumDashValidator.create(TRegex.create()),
    TMinLengthValidator.create(8),
    TMaxLengthValidator.create(10),
]);
```

Please note that order of validator in array matters as validation is carried out starting from first validation rule in the array and to next rule. As soon as validator fails, next validator will not be called and error message reported is last failed validation rule error message.

### TCollectiveValidator

It is similar to `TCompositeValidator`, except that, instead of terminating validation process as soon as one of validation rule fails, it continues to process all validation rules and collects all failed validation rule messages.

```
rule := TCollectiveValidator.create([
    TRequiredValidator.create(),
    TAlphaNumDashValidator.create(TRegex.create()),
    TMinLengthValidator.create(8),
    TMaxLengthValidator.create(10),
]);
```

### TNotValidator

Decorator validator class which negate other validator.

```
rule := TNotValidator.create(
    'Field %s must not be integer value',
    TIntegerValidator.create()
);
```
Validation will pass as long as data is not integer value. To negate several validator,
you can combine with `TCompositeValidator`.

```
rule := TNotValidator.create(
    'Field %s must not be integer or boolean value',
    TCompositeValidator.create([
        TIntegerValidator.create(),
        TBooleanValidator.create()
    ])
);
```
Validation will pass as long as data is not integer value and not boolean value.

### TConfirmedValidator

Data must equal to other predefined field. This is mostly to be used for password confirmation.

```
rule := TConfirmedValidator.create('password_confirmation');
```
Validation will pass if current data is equal to data in field `password_confirmation`.

### TUuidValidator

Data must be in valid GUID string.

```
rule := TUuidValidator.create();
```

### TBaseValidator

Base abstract class which can be use as a base class to create new custom validation rule.

## Explore more

- [Form Validation](/security/form-validation)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
