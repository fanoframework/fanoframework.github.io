---
title: Built-in validation rules
description: List of built-in validation rules in Fano Framework
---

<h1 class="major">Built-in Validation Rules</h1>

This page lists all available built-in validation rules that Fano Framework provides. For information on how to use these validation rules in application, please read [Form Validation](/security/form-validation) documentation.

| [Alpha](#talphavalidator) | [Alpha Num](#talphanumvalidator) | [Alpha num space](#talphanumspacevalidator) |
| [Required](#trequiredvalidator) | [Present](#tpresentvalidator) | [Required if](#trequiredifvalidator) |
| [Alpha num dash](#talphanumdashvalidator) | [Email](#temailvalidator) | [Url](#turlvalidator) |
| [Slug](#tslugvalidator) | [Phone](#tphonevalidator) | [Regex](#tregexvalidator) |
| [Equal string](#tequalstrvalidator) | [Case insensitive equal string](#tcaseinsensitiveequalstrvalidator) | [Equal length](#tequallengthvalidator) |
| [Min length](#tminlengthvalidator) | [Max length](#tmaxlengthvalidator) | [Integer](#tintegervalidator) |
| [Int64](#tint64validator) | [Dword](#tdwordvalidator) | [Qword](#tqwordvalidator) |
| [Float](#tfloatvalidator) | [Currency](#tcurrencyvalidator) | [Numeric](#tnumericvalidator) |
| [Min integer](#tminintegervalidator) | [Max integer](#tmaxintegervalidator) | [Less than](#tlessthanvalidator) |
| [Greater than](#tgreaterthanvalidator) | [Equal integer](#tequalintvalidator) | [Positive integer](#tpositiveintvalidator) |
| [Negative integer](#tnegativeintvalidator) | [Odd integer](#toddintvalidator) | [Even integer](#tevenintvalidator) |
| [Boolean](#tbooleanvalidator) | [Accepted](#tacceptedvalidator) | [In](#tinvalidator) |
| [In integer](#tinintvalidator) | [Not in integer](#tnotinintvalidator) | [Date](#tdatevalidator) |
| [Date time](#tdatetimevalidator) | [Time](#ttimevalidator) | [Equal date time](#tequaldatetimevalidator) |
| [After date time](#tafterdatetimevalidator) | [Before date time](#tbeforedatetimevalidator) | [After date time field](#tafterdatetimefieldvalidator) |
| [Before date time field](#tbeforedatetimefieldvalidator) | [File](#tfilevalidator) | [Directory](#tdirectoryvalidator) |
| [Uploaded file](#tuploadedfilevalidator) | [Uploaded Mime](#tuploadedmimevalidator) | [Uploaded size](#tuploadedsizevalidator) |
| [File format](#tfileformatvalidator) | [Image PNG](#timagepngvalidator-timagejpgvalidator-timagegifvalidator) | [Image JPG](#timagepngvalidator-timagejpgvalidator-timagegifvalidator) |
| [Image GIF](#timagepngvalidator-timagejpgvalidator-timagegifvalidator) | [Anti virus](#tantivirusvalidator) | [Composite](#tcompositevalidator) |
| [Collective](#tcollectivevalidator) | [Or](#torvalidator) | [Any of](#tanyofvalidator) |
| [And](#tandvalidator) | [Not](#tnotvalidator) | [Confirmed](#tconfirmedvalidator) |
| [Same](#tsamevalidator) | [UUID](#tuuidvalidator) | [Exists](#texistsvalidator) |
| [Always pass](#talwayspassvalidator) | [IPv4](#tipv4validator) | [IPv6](#tipv6validator) |
| [MAC](#tmacaddrvalidator) | [JSON](#tjsonvalidator) | [Base64](#tbase64validator) |
| [Color](#tcolorvalidator) | [Start With](#tstartwithvalidator) | [End With](#tendwithvalidator) |
| [Latitude](#tlatitudevalidator) | [Longitude](#tlongitudevalidator) | [Between](#tbetweenvalidator) |
| [Less or equal than](#tlessorequalthanvalidator) | [Greater or equal than](#tgreaterorequalthanvalidator) | - |

## Field availability

Following validators test if field is present.

### <a name="trequiredvalidator"></a>TRequiredValidator

Field must be present and not empty.

```
var rule : IValidator;
...
rule := TRequiredValidator.create();
```

### <a name="tpresentvalidator"></a>TPresentValidator

Field must be present but allowed to be empty.

```
rule := TPresentValidator.create();
```

### <a name="trequiredifvalidator"></a>TRequiredIfValidator

Field must be present and not empty if other field passes validation.

```
var rule : IValidator;
...
rule := TRequiredIfValidator.create(
    'other-field',
    TInValidator.create(['foo', 'bar'])
);
```
Field being validated becomes mandatory field only if `other-field` value is equal to `foo` or `bar`.

Complex rule can be achieved by composing several validation rules.

```
var rule : IValidator;
...
rule := TRequiredIfValidator.create(
    'other-field',
    TCompositeValidator.create([
        TNotInValidator.create(['foo', 'bar'])
        TMinLengthValidator.create(8)
    ])
);
```

Field being validated becomes mandatory field, only if `other-field` value is not equal to `foo` or `bar` and at least 8 characters.

## Character format

### <a name="talphavalidator"></a>TAlphaValidator

Data must be alphabet character only. This class expect instance of `IRegex` interface.

```
rule := TAlphaValidator.create(TRegex.create());
```

### <a name="talphanumvalidator"></a>TAlphaNumValidator

Data must be alphabet and numeric character only.

```
rule := TAlphaNumValidator.create(TRegex.create());
```

### <a name="talphanumspacevalidator"></a>TAlphaNumSpaceValidator

Data must be alphabet, numeric and space character only.

```
rule := TAlphaNumSpaceValidator.create(TRegex.create());
```

### <a name="talphanumdashvalidator"></a>TAlphaNumDashValidator

Data must be alphabet, numeric and dash character only.

```
rule := TAlphaNumDashValidator.create(TRegex.create());
```

### <a name="temailvalidator"></a>TEmailValidator

Data must be in valid email format. This validator does not check if email address actually exists.

```
rule := TEmailValidator.create(TRegex.create());
```

### <a name="turlvalidator"></a>TUrlValidator

Data must be in valid URL format. This validator does not check if URL address actually exists.

```
rule := TUrlValidator.create(TRegex.create());
```

### <a name="tslugvalidator"></a>TSlugValidator

Data must be in slug format.

```
rule := TSlugValidator.create(TRegex.create());
```

This validator will give result as shown in following example input,

```
'my-slug-title' ==> pass
'my-slug-title-1' ==> pass
'1-my-slug-title' ==> pass
'-my-slug-title' ==> fail
'my-slug-title-' ==> fail
'my-slug--title' ==> fail
```

### <a name="tphonevalidator"></a>TPhoneValidator

Data must be in phone number format.

```
rule := TPhoneValidator.create(TRegex.create());
```

This validator will give result as shown in following example input,

```
'081123456' ==> pass
'081-1234-56-7' ==> pass
'(081)-1234-56-7' ==> pass
'081-12-34-567' ==> pass
'a0811234567' ==> fail
'081-1234-56-7a' ==> fail
```
### <a name="tbase64validator"></a>TBase64Validator

Data must be in Base64-encoded string format.

```
rule := TBase64Validator.create();
```

### <a name="tstartwithvalidator"></a>TStartWithValidator

Data must start with a string value.

```
rule := TStartWithValidator.create('Hello');
```
This validator will give result as shown in following example input,

```
'Hello' ==> pass
'HelloWorld' ==> pass
'Hello Hello' ==> pass
'myHello' ==> fail
'hello' ==> fail
'hello world' ==> fail
```
### <a name="tendwithvalidator"></a>TEndWithValidator

Data must end with a string value.

```
rule := TEndWithValidator.create('Hello');
```
This validator will give result as shown in following example input,

```
'Hello' ==> pass
'myHello' ==> pass
'Hello Hello' ==> pass
'Hello ' ==> fail
'Hello world' ==> fail
'hello world' ==> fail
```

### <a name="tregexvalidator"></a>TRegexValidator

Data must be in format as specified by regular expression pattern.

```
rule := TRegexValidator.create(
    TRegex.create(),
    '[a-zA-Z]+',
    'Field %s does not match regular expression'
);
```
Parameters are `IRegex` instance, regular expression pattern and error message.
`%s` will be replaced with field name being validated.

## Constant string equality

### <a name="tequalstrvalidator"></a>TEqualStrValidator

Data must match predefined constant string. For example, data being validated must equals string `test` to pass validation.

```
rule := TEqualStrValidator.create('test');
```

### <a name="tcaseinseinsitiveequalstrvalidator"></a>TCaseInsensitiveEqualStrValidator

Data must match predefined constant string in case-insensitive manner. For example, data being validated can equal to `test`, `Test`, `teSt` to pass validation.

```
rule := TCaseInsensitiveEqualStrValidator.create('test');
```

## String length

### <a name="tequallengthvalidator"></a>TEqualLengthValidator

Data must be string with its length is equal to predefined value. For example, data being validated must contains exactly 10 characters to pass validation.

```
rule := TEqualLengthValidator.create(10);
```

### <a name="tminlengthvalidator"></a>TMinLengthValidator

Data must be string with its length not less than predefined value.

```
rule := TMinLengthValidator.create(10);
```

### <a name="tmaxlengthvalidator"></a>TMaxLengthValidator

Data must be string with its length not greater than predefined value.

```
rule := TMaxLengthValidator.create(10);
```

## Numeric data

### <a name="tintegervalidator"></a>TIntegerValidator

Data must be integer value.

```
rule := TIntegerValidator.create();
```

### <a name="tint64validator"></a>TInt64Validator

Data must be int64 value.

```
rule := TInt64Validator.create();
```
### <a name="tdwordvalidator"></a>TDwordValidator

Data must be dword value.

```
rule := TDwordValidator.create();
```
### <a name="tqwordvalidator"></a>TQwordValidator

Data must be qword value.

```
rule := TQwordValidator.create();
```

### <a name="tfloatvalidator"></a>TFloatValidator

Data must be float value.

```
rule := TFloatValidator.create();
```

### <a name="tcurrencyvalidator"></a>TCurrencyValidator

Data must be currency value.

```
rule := TCurrencyValidator.create();
```

### <a name="tnumericvalidator"></a>TNumericValidator

Data must be integer or float value.

```
rule := TNumericValidator.create();
```

### <a name="tminintegervalidator"></a>TMinIntegerValidator

Data must be integer value and its value not less than predefined value.

```
rule := TMinIntegerValidator.create(100);
```
Validation will pass only if you pass integer value >= 100.

### <a name="tmaxintegervalidator"></a>TMaxIntegerValidator

Data must be integer value and not greater than predefined value.

```
rule := TMaxIntegerValidator.create(100);
```
Validation will pass only if you pass integer value <= 100.

### <a name="tlessthanvalidator"></a>TLessThanValidator, TFloatLessThanValidator, TCurrLessThanValidator

Data must be integer value less than predefined value.

```
rule := TLessThanValidator.create(100);
```
Validation will pass only if you pass integer value < 100.

To validate floating-point or currency value use `TFloatLessThanValidator` or `TCurrLessThanValidator` respectively as shown below.

```
rule := TFloatLessThanValidator.create(100.5);
```
Validation will pass only if you pass float value less than 100.5.

### <a name="tlessorequalthanvalidator"></a>TLessOrEqualThanValidator, TFloatLessOrEqualThanValidator, TCurrLessOrEqualThanValidator

Similar to `TLessThanValidator` above except it uses <= comparison.

### <a name="tgreaterthanvalidator"></a>TGreaterThanValidator, TFloatGreaterThanValidator, TCurrGreaterThanValidator

Data must be integer value greater than predefined value.

```
rule := TGreaterThanValidator.create(100);
```
Validation will pass only if you pass integer value > 100.

To validate floating-point or currency value use `TFloatGreaterThanValidator` or `TCurrGreaterThanValidator` respectively as shown below.

```
rule := TFloatGreaterThanValidator.create(100.5);
```
Validation will pass only if you pass float value greater than 100.5.

### <a name="tgreaterorequalthanvalidator"></a>TGreaterOrEqualThanValidator, TFloatGreaterOrEqualThanValidator, TCurrGreaterOrEqualThanValidator

Similar to `TGreaterThanValidator` above except it uses >= comparison.

### <a name="tbetweenvalidator"></a>TBetweenValidator, TFloatBetweenValidator, TCurrBetweenValidator

Data must be integer value btween predefined low and high value.

```
rule := TBetweenValidator.create(50, 100);
```
Validation will pass only if you pass integer value >=50 and <= 100.

To validate floating-point or currency value use `TFloatBetweenValidator` or `TCurrBetweenValidator` respectively as shown below.

```
rule := TFloatBetweenValidator.create(50.5, 100.5);
```
Validation will pass only if you pass float value value >=50.5 and <= 100.5.

### <a name="tequalintvalidator"></a>TEqualIntValidator

Data must be integer value equals predefined value.

```
rule := TEqualIntValidator.create(100);
```
Validation will pass only if you pass integer equals 100.

### <a name="tpositiveintvalidator"></a>TPositiveIntValidator

Data must be positive integer value.

```
rule := TPositiveIntValidator.create();
```

### <a name="tnegativeintvalidator"></a>TNegativeIntValidator

Data must be negative integer value.

```
rule := TNegativeIntValidator.create();
```

### <a name="toddintvalidator"></a>TOddIntValidator

Data must be odd integer value.

```
rule := TOddIntValidator.create();
```

### <a name="tevenintvalidator"></a>TEvenIntValidator

Data must be even integer value.

```
rule := TEvenIntValidator.create();
```

## Boolean data

### <a name="tbooleanvalidator"></a>TBooleanValidator

Data must be boolean value

```
rule := TBooleanValidator.create();
```

### <a name="tacceptedvalidator"></a>TAcceptedValidator

Data must be boolean value `true`, or integer 1 or string `yes` or string `on`. This is mostly to be used for accepting "Terms and Conditions".

```
rule := TAcceptedValidator.create();
```

## Ranged data

### <a name="tinvalidator"></a>TInValidator

Data must be one of predefined string values.

```
rule := TInValidator.create(['foo', 'bar']);
```
Validation will pass only if its value equals to `foo` or `bar`.

### <a name="tinintvalidator"></a>TInIntValidator

Data must be one of predefined integer values.

```
rule := TInIntValidator.create([1, 2, 3]);
```
Validation will pass only if its value equals to 1, 2 or 3.


### <a name="tnotinvalidator"></a>TNotInValidator

Data must not be one of predefined values.

```
rule := TNotInValidator.create(['foo', 'bar']);
```
Validation will pass only if its value is not equal to `foo` or `bar`.

## Date and time

### <a name="tdatevalidator"></a>TDateValidator

Data must be valid date

```
rule := TDateValidator.create();
```

### <a name="tdatetimevalidator"></a>TDateTimeValidator

Data must be valid date time.

```
rule := TDateTimeValidator.create();
```

### <a name="ttimevalidator"></a>TTimeValidator

Data must be valid time.

```
rule := TTimeValidator.create();
```

### <a name="tequaldatetimevalidator"></a>TEqualDateTimeValidator

Data must be date time and its value must equal predefined value.

```
rule := TEqualDateTimeValidator.create(date());
```

### <a name="tafterdatetimevalidator"></a>TAfterDateTimeValidator

Data must be date time and its value must be later than predefined value.

```
rule := TAfterDateTimeValidator.create(date());
```

### <a name="tbeforedatetimevalidator"></a>TBeforeDateTimeValidator

Data must be date time and its value must be prior than predefined value.

```
rule := TBeforeDateTimeValidator.create(date());
```

### <a name="tafterdatetimefieldvalidator"></a>TAfterDateTimeFieldValidator

This validation rule is similar to `TAfterDateTimeValidator` but it compares against other field instead of predefined value.

```
rule := TAfterDateTimeFieldValidator.create('event-start-date');
```

Above rule makes current field passes validation only if its value is later than the value in `event-start-date` field.

### <a name="tbeforedatetimefieldvalidator"></a>TBeforeDateTimeFieldValidator

This validation rule is similar to `TBeforeDateTimeValidator` but it compares against other field instead of predefined value.

```
rule := TBeforeDateTimeFieldValidator.create('event-end-date');
```

Above rule makes current field passes validation only if its value is prior than the value in `event-end-date` field.

## <a name="file-directory"></a>File and Directory

### <a name="tfilevalidator"></a>TFileValidator

Data must be name of file which exists.

```
rule := TFileValidator.create('images/image.jpg');
```

### <a name="tdirectoryvalidator"></a>TDirectoryValidator

Data must be name of directory which exists.

```
rule := TDirectoryValidator.create('images');
```

## <a name="uploaded-file"></a>Uploaded file

### <a name="tuploadedfilevalidator"></a>TUploadedFileValidator

Field must be valid uploaded file.

```
rule := TUploadedFileValidator.create();
```

### <a name="tuploadedmimevalidator"></a>TUploadedMimeValidator

Field must be valid uploaded file and its content type must be one of predefined MIME types.

```
rule := TUploadedMimeValidator.create(['image/jpg', 'image/png']);
```
Please note that this validation rule does not check if file uploaded is actually JPEG/PNG image.

### <a name="tuploadedsizevalidator"></a>TUploadedSizeValidator

Field must be valid uploaded file and its file size must be less or equal predefined size in bytes. For example to restrict that uploaded file size must not exceed 200KB.

```
rule := TUploadedSizeValidator.create(200 * 1024);
```

### <a name="tfileformatvalidator"></a>TFileFormatValidator

Abstract class which validate uploaded file against certain file format.

### <a name="timagepngvalidator-timagejpgvalidator-timagegifvalidator"></a>TImagePngValidator, TImageJpgValidator, TImageGifValidator

Validation rules inherit from `TFileFormatValidator` which validate uploaded file against PNG, JPEG and GIF image format. Please note these classes only do fast check on first few bytes of file to determine its format and without loading whole file.

Validate field that must be uploaded file with PNG image format
```
rule := TImagePngValidator.create();
```

Validate field that must be uploaded file with JPEG image format

```
rule := TImageJpgValidator.create();
```

Validate field that must be uploaded file with GIF image format

```
rule := TImageGifValidator.create();
```

### <a name="tantivirusvalidator"></a>TAntivirusValidator

Field must be valid uploaded file and must be free from computer virus. Current implementation supports [ClamAV](https://www.clamav.net) via [clamav-daemon](https://www.clamav.net/documents/scanning#clamd) and null implementation only.

#### ClamAV
Previous implementation which uses [libclamav](https://www.clamav.net/documents/libclamav)  directly is removed due to unstable code.

To use ClamAV, you need to install ClamAV antivirus and its daemon on server. For example

```
$ sudo apt-get install clamav clamav-daemon
```
Run `clamav-daemon` service,

```
$ sudo systemctl start clamav-daemon
```

Create validator as follows,

```
rule := TAntivirusValidator.create(
    TLocalClamdAv.create('/var/run/clamav/clamd.ctl')
);
```

where `/var/run/clamav/clamd.ctl` is default unix domain socket file where `clamav-daemon` is listening. To listen using TCP socket, use,

```
rule := TAntivirusValidator.create(
    TLocalClamdAv.create('127.0.0.1', 3310)
);
```

`TLocalClamdAv` works by sending absolute file path. It assumes that ClamAV daemon is running on same machine as our application, thus, can read uploaded file directly.

If daemon is running on different machine than application, it can not read file directly, so you can not use `TLocalClamdAv` and must use `TClamdAv`. `TClamdAv` works by sending content of file to daemon, thus more expensive in term of network operation but more flexible as it allows daemon to run on different machine than application.

```
rule := TAntivirusValidator.create(
    TClamdAv.create('/var/run/clamav/clamd.ctl')
);
```

Listening on TCP socket by default is disabled in configuration. You can inspect current ClamAV configuration by running `clamconf` command. Read [ClamAV configuration](https://www.clamav.net/documents/configuration) for information on how to configure it.

To safely test antivirus validator, use [Eicar virus sample test file](http://www.eicar.org/download/eicar.com.txt). It does not contain actual virus so it is safe, but many anti virus vendors agree to report it as virus. ClamAV will report it with virus signature `Eicar-Test-Signature`. [Read Eicar test file for more information](https://en.wikipedia.org/wiki/EICAR_test_file).

Please note that virus scanning is expensive task especially if you handle big file.
You should think carefully about performance when using this validator.

#### Null
To use null implementation, use `TNullAv` class.

```
rule := TAntivirusValidator.create(TNullAv.create());
```
This is provided to bypass antivirus scanning.

## Miscellaneous

### <a name="tcompositevalidator"></a>TCompositeValidator

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

### <a name="tcollectivevalidator"></a>TCollectiveValidator

It is similar to `TCompositeValidator`, except that, instead of terminating validation process as soon as one of validation rule fails, it continues to process all validation rules and collects all failed validation rule messages.

```
rule := TCollectiveValidator.create([
    TRequiredValidator.create(),
    TAlphaNumDashValidator.create(TRegex.create()),
    TMinLengthValidator.create(8),
    TMaxLengthValidator.create(10),
]);
```

### <a name="torvalidator"></a>TOrValidator

It is similar to `TCollectiveValidator`, except that, it uses boolean operator OR when test each validator, thus it returns false only when all external validators are failed and returns true if one or more validators are succeeded.

```
rule := TOrValidator.create([
    TRequiredValidator.create(),
    TAlphaNumDashValidator.create(TRegex.create()),
    TMinLengthValidator.create(8),
    TMaxLengthValidator.create(10),
]);
```

### <a name="tanyofvalidator"></a>TAnyOfValidator

Alias of `TOrValidator`.

### <a name="tandvalidator"></a>TAndValidator

It is alias of `TCompositeValidator`.

```
rule := TAndValidator.create([
    TRequiredValidator.create(),
    TAlphaNumDashValidator.create(TRegex.create()),
    TMinLengthValidator.create(8),
    TMaxLengthValidator.create(10),
]);
```

### <a name="tnotvalidator"></a>TNotValidator

Decorator validator class which negate other validator.

```
rule := TNotValidator.create(
    'Field %s must not be integer value',
    TIntegerValidator.create()
);
```
Validation will pass as long as data is not integer value. To negate several validators,
you can combine them with `TCompositeValidator`.

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

### <a name="tconfirmedvalidator"></a>TConfirmedValidator

Data must equal to other predefined field. This is mostly to be used for password confirmation.

```
rule := TConfirmedValidator.create('password_confirmation');
```
Validation will pass if current data is equal to data in field `password_confirmation`.

### <a name="tsamevalidator"></a>TSameValidator

This is alias for `TConfirmedValidator`. It passes validation if other field has same value as field being validated.

```
rule := TSameValidator.create('password_confirmation');
```

### <a name="tuuidvalidator"></a>TUuidValidator

Data must be in valid GUID string.

```
rule := TUuidValidator.create();
```

### <a name="texistsvalidator"></a>TExistsValidator

Data must exist in database. Its constructor expects three parameters, instance of `IRdbms` interface representing database connection, table name and column name.

```
var db : IRdbms;
...
db := TMySQLDb.create('MySQL 5.7');
db.connect('localhost', 'your_db_name', 'your_db_username', 'yoursecretpassword', 3306);
rule := TExistsValidator.create(db, 'table_users', 'user_id');
```

Above code will actually execute SQL as follows

```
SELECT `user_id` FROM `table_users` WHERE user_id = :primaryId LIMIT 1
```
where `primaryId` placeholder will be replace with actual data being validated. Read [Working with Database](/database) for more information.

### <a name="talwayspassvalidator"></a>TAlwaysPassValidator

Dummy validator which always make field passes validation.

```
rule := TAlwaysPassValidator.create();
```

### <a name="tipv4validator"></a>TIpv4Validator

Data must be valid IP address (IPv4).

```
rule := TIpv4Validator.create();
```

### <a name="tipv6validator"></a>TIpv6Validator

Data must be valid IP address (IPv6).

```
rule := TIpv6Validator.create();
```

### <a name="tmacaddrvalidator"></a>TMacAddrValidator

Data must be valid MAC address.

```
rule := TMAcAddrValidator.create(TRegex.create());
```

### <a name="tjsonvalidator"></a>TJsonValidator

Data must be valid JSON.

```
rule := TJsonValidator.create();
```

### <a name="tcolorvalidator"></a>TColorValidator

Data must be valid hex color value such as `#ffffff` or `#fff`.

```
rule := TColorValidator.create(TRegex.create());
```

### <a name="tlatitudevalidator"></a>TLatitudeValidator

Data must be valid latitude float value.

```
rule := TLatitudeValidator.create();
```
### <a name="tlongitudevalidator"></a>TlongitudeValidator

Data must be valid longitude float value.

```
rule := TLongitudeValidator.create();
```

### TBaseValidator

Abstract class which can be uses as a base class to create new custom validation rule. To inherit from this class, you need to implements its `isValidData()` method.

```
function isValidData(
    const dataToValidate : string;
    const dataCollection : IList;
    const request : IRequest
) : boolean; virtual; abstract;
```

- `dataToValidate` will contain actual data of current field being validated.
- `dataCollection` will contain list of key value pair of all query strings and POST body of request.
- `request` will contain current request object, in case you need to more informations.

Your implementation must return `true` if validation should pass or `false` otherwise.

### TCompareFieldValidator

Abstract class which can be used as base class which has the ability to compare current field against other field. To inherit from this class, you need to implements its `compare()` method.

```
function compare(
    const dataToValidate : string;
    const otherFieldData : string
) : boolean; virtual; abstract;
```
- `dataToValidate` will contain actual data of current field being validated.
- `dataCollection` will contain actual data of other field being compared.

Your implementation must return `true` if validation should pass or `false` otherwise.

## Explore more

- [Form Validation](/security/form-validation)
