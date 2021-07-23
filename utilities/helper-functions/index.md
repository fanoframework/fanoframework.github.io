---
title: Helper functions
description: Tutorial on how to use helper functions provided by Fano Framework
---

<h1 class="major">Helper functions</h1>

## About Helper function

![Helper illustration](/assets/images/helping-hand.jpg){:class="image fit"}
(Photo by [Austin Kehmeier](https://unsplash.com/@a_kehmeier?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/helping-hand?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText))

Fano Framework provides helper functions to simplify common tasks such as [generate slug string](#generate-slug-string), [join string with delimiter](#join-string) or [read file content](#read-file-content).

All helpers functions are declared in `fano.pas` unit.

## <a name="generate-slug-string"></a>Read file content to string

`readFile()` reads file content and output it as string. It accepts one parameter file path of filename to load.

```
(*!------------------------------------------------
 * read file to string
 *-----------------------------------------------
 * @param filepath file to load
 * @returns contents of file as string
 *-----------------------------------------------*)
function readFile(const filepath : string) : string;
```

Please note, it may consume a lot of memories if file size is big.

```
var fileContent : string;
...

fileContent := readFile('/tmp/myfile');
```

## <a name="join-string"></a>Join array of string as string

`join()` function concatenates array of string as a single string with delimiter.

```
(*!------------------------------------------------
 * join array of string with a delimiter
 *-----------------------------------------------
 * @param delimiter string to separate each
 * @param values array of string to join
 *-----------------------------------------------*)
function join(const delimiter : string; const values : array of string) : string;
```

For example
```
var joinedStr : string;
...

//joinedStr = 'hello#good#people';
joinedStr := join('#', ['hello', 'good', 'people']);
```

## <a name="generate-slug-string"></a>Generate slug

`slug()` function generates string consists of alpha numeric characters separated by dash characters and strips everything else.

```
(*!------------------------------------------------
 * generate slug
 *-----------------------------------------------
 * @param originalStr input string
 * @return slug
 *-----------------------------------------------*)
function slug(const originalStr : string) : string;
```

For example

```
var slugStr : string;
...

//slugStr = 'hello-good-people';
slugStr := slug('hello    good  @##@$$@$people']);
```

## Explore more

- [Regular expression](/utilities/regular-expression)
