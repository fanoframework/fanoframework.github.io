---
title: Built-in validation rules
description: List of built-in validation rules in Fano Framework
---

<h1 class="major">Built-in Validation Rules</h1>

## Field  availability

Following validator test if field is present.
### TRequiredValidator

Field must be present and not empty.

### TPresentValidator

Field must be present but allowed to be empty.

## Character format

### TAlphaValidator

Data must be alphabet character only.

### TAlphaNumValidator

Data must be alphabet and numeric character only.

### TAlphaNumSpaceValidator

Data must be alphabet, numeric and space character only.

### TAlphaNumDashValidator

Data must be alphabet, numeric and dash character only.

### TEmailValidator

Data must be in valid email format. This validator is not check if email address actually exists.

### TUuidValidator

Data must be in valid GUID string.

### TRegexValidator

Data must be in format as specified by regular expression pattern.

## String length

### TEqualLengthValidator

Data must be string with its length equal predefined value.

### TMinLengthValidator

Data must be string with its length not less than predefined value.

### TMaxLengthValidator

Data must be string with its length not greter than predefined value.

## Numeric data

### TIntegerValidator

Data must be integer value.

### TFloatValidator

Data must be float value.

### TCurrencyValidator

Data must be currency value.

### TNumericValidator

Data must be integer or float value.

### TMinIntegerValidator

Data must be integer value and not less than predefined value.

### TMaxIntegerValidator

Data must be integer value and not greater than predefined value.

## Boolean data

### TBooleanValidator

Data must be boolean value

### TAcceptedValidator

Data must be boolean value `true`, or integer 1 or string `yes` or string `on`. This is mostly to be used for accepting "Terms and Conditions".

## Ranged data

### TInValidator

Data must be one of predefined values.

### TNotInValidator

Data must not be one of predefined values.

## Date and time

### TDateValidator

Data must be valid date

### TDateTimeValidator

Data must be valid date time.

### TTimeValidator

Data must be valid time.

### TEqualDateTimeValidator

Data must be date time and its value must equal predefined value.

### TAfterDateTimeValidator

Data must be date time and its value must be later than predefined value.

### TBeforeDateTimeValidator

Data must be date time and its value must be prior than predefined value.

## Uploaded file

### TUploadedFileValidator

Field must be valid uploaded file.

### TUploadedMimeValidator

Field must be valid uploaded file and its content type must be one of predefined MIME types.

### TUploadedSizeValidator

Field must be valid uploaded file and its file size must be less or equal predefined size in bytes.

## Miscellaneous

### TCompositeValidator

Validator class which combined one or more validators as one.

### TNotValidator

Decorator validator class which negate other validator.

### TConfirmedValidator

Data must equal to other predefined field. This is mostly to be used for password confirmation.

### TBaseValidator

Base abstract class which  can be use as a base class for validation rule.

## Explore more

- [Form Validation](/security/form-validation)

<ul class="actions">
    <li><a href="/documentation" class="button">Documentation</a></li>
</ul>
