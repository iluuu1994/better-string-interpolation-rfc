# PHP RFC: Better string interpolation

* Date: 2020-08-16
* Author: Ilija Tovilo, ilutov@php.net
* Status: Draft
* Target Version: PHP 8.1
* Implementation: Pending

## Introduction

This RFC proposes to introduce a new type of string interpolation `$"#{expr}"` for arbitrary expressions.

## Proposal

PHP already has several types of string interpolation.

1. Directly embedding variables (`"$foo"`)
2. Braces outside the variable (`"{$foo}"`)
3. Braces after the dollar sign (`"${foo}"`)
4. Dynamic variable lookup (`"${expr}"`)

## Contexts

Currently, string interpolation works in the following contexts:

1. Double quoted strings (`""`)
2. Heredocs (`<<<FOO ... FOO`)
3. Backticks (\`\`)

The new string interpolation is supported for each of these contexts.

```php
$x = $"Foo {$bar}";
```

## Other languages

## Backward Incompatible Changes

There are no known backward incompatible changes in this RFC.

## Vote

...











Hi internals

I've been thinking about ways to improve string interpolation. String
interpolation in PHP is currently pretty limited to say the least.
What's worse is we have several ways to do the same thing and a few of
them being quite inconsistent and confusing.

We have these syntaxes:

1. Directly embedding variables: "$foo"
2. Braces outside the variable "{$foo}"
3. Braces after the dollar sign: "${foo}"
4. Dynamic variable lookup: "${expr}"

The difference between 3 and 4 is extra confusing. Passing a simple
variable name without a dollar inside the braces (and optionally an
offset access) will embed the given variable in the string. When
passing a more complex expression ${} will behave completely
differently and perform a dynamic variable lookup.
https://3v4l.org/uqcjf

Let's look at some different examples.

Simple local variables work well, although it is questionable why we
have 4 ways to do the same thing.

    "$foo"
    "{$foo}"
    "${foo}"
    "${'foo'}"

Accessing offsets is supported for syntax 1, 2 and 3. String quoting
in the offset expression is inconsistent.

    "$foo[bar]"
    "{$foo['bar']}"
    "${foo['bar']}"

Accessing properties is supported for syntax 1 and 2.

    "$foo->bar"
    "{$foo->bar}"

Nested property fetches or offset accesses only work with syntax 2.

    "$foo[bar][baz]" // [baz] is not part of the expression, equivalent
    to "{$foo['bar']}[baz]"
    "{$foo['bar']['baz']}"
    "${foo['bar']['baz']}" // Syntax error

Calling methods is only supported for syntax 2.

    "$foo->bar()" // Treats ->bar as a property access, equivalent to
    "{$foo->bar}()"
    "{$foo->bar()}"

Arbitrary expressions work for none of the syntaxes.

        "{$foo + 2}" // Syntax error
        "{Foo::bar()}" // Interpreted as string

The most functional of these syntaxes is 2 but even here only a small
subset of expressions are accepted. The distinction between syntax 3
and 4 is very confusing and we should probably deprecate and remove
syntax 3 (as it does pretty much the same as syntax 2). Sadly, the
only syntax that accepts arbitrary expressions (4) is also the least
useful.

As to allowing arbitrary string interpolation, we have three options:

    Deprecate syntax 4, remove it, and change its functionality in the future
    Use some new syntax (e.g. "(1 + 2)", "{1 + 2}", "$(1 + 2)", etc)
    with the given BC break
    Do nothing

Adding new syntax will break many regular expressions and embedded
jQuery code, although they are easily fixed by escaping the \ or $.
Deprecating, removing and then changing ${} would take many years. I
don't know which of these is the better approach.

Any thoughts or different ideas?

Ilija
