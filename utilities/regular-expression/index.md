---
title: Working with regular expression
description: Tutorial on how to use regex utility in Fano Framework
---

<h1 class="major">Working with regular expression</h1>

## Regex utility

Fano Framework provides regex utility through `IRegex` interface.

```
IRegex = interface
    ['{E08AD12B-C606-48FF-A9FA-728EAB14AB35}']
    function replace(
        const regexPattern : string;
        const source : string;
        const replacement : string
    ) : string;

    function quote(const regexPattern : string) : string;

    function match(
        const regexPattern : string;
        const source : string
    ) : TRegexMatchResult;

    function greedyMatch(
        const regexPattern : string;
        const source : string
    ) : TRegexMatchResult;

end;
```
## Built-in IRegex implementation

Currently, Fano Framework provides `TRegex` class that implements `IRegex` interface which is a thin wrapper for Free Pascal built-in regex library ([TRegExpr](https://regex.sorokin.engineer/en/latest/)).

To create different `IRegex` implementation, just create new class that implement this interface.

## Match string against regex pattern

To match string against regex pattern, you use `match()` or `greedyMatch()` method.
Former method stops as soon as it finds a match while latter try to match as many as possible.

They accept same parameters:

- `regexPattern` contains regular expression pattern to match.
- `source` contains actual string to match.

They return `TRegexMatchResult` which is a record declared like so

```
TRegexMatches = array of array of string;
TRegexMatchResult = record
    matched : boolean;
    matches : TRegexMatches;
end;
```

`matched` field tells if there is a match and matched strings are in `matches` field which is empty array when `matched`is false.

```
var
    regex : IRegex;
    matches : TRegexMatchResult;
...
regex := TRegex.create();
matches := regex.match('\[\[\s*([a-zA-Z0-9]*)\s*\]\]', 'This is [[ myVar ]]');
```
Above code matches string inside `[[ ... ]]` and will yield

```
matches.matched = true
matches.matches[0][0] = '[[ myVar ]]'
matches.matches[0][1] = 'myVar'
```

## Replace string with regex pattern.

To replace string call `replace()` method. It accepts three parameters,

- `regexPattern` contains regex pattern to match.
- `source` is input string
- `replacement` is replacement

When it finds match, `replacement` wil replace matched string.

```
var
    regex : IRegex;
    newStr : string;
...
regex := TRegex.create();
newStr := regex.match('\[\[\s*([a-zA-Z0-9]*)\s*\]\]', 'This is [[ myVar ]]', 'cool');
```
Above code matches string inside `[[ ... ]]` and will yield

```
newStr = 'This is cool'
```

# Explore more

- [TRegex class](https://github.com/fanoframework/fano/blob/master/src/Libs/Regex/RegexImpl.pas)
- [IRegex interface](https://github.com/fanoframework/fano/blob/master/src/Libs/Regex/Contracts/RegexIntf.pas)
