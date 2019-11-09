# `insertionsort`: sort lists using a primitive insertionsort

Author
: Jonathan P. Spratte

License
: [LPPL v1.3c](https://www.latex-project.org//lppl/lppl-1-3c.txt) or later

Copyright (C)
: 2019

## About

`insertionsort` is a small project aiming to add a list sorting implementation
that is lightweight and fast (on smaller lists).

## Status

`insertionsort` is somewhat usable. The current insertionsort-implementation
simply compares the new value with each value in the sorted list in order. It is
therefore fastest for reverse-sorted input, and slowest for correctly sorted
input. Currently there is no smart binary searching or something like that.

As of now, I haven't decided on the final implementation, hence two versions are
provided:

[`insertionsort_complicated`](#complicated)
: this version is faster for lists in which each element's value has to be
calculated, e.g., your list contains vectors and you want to sort them by their
lengths. It uses more tokens in a temporary macro.

[`insertionsort_simple`](#simple)
: this version is faster for lists in which each element is the value, e.g., a
list of integers.

You can use both versions in parallel.

## Short Documentation

Each version has a macro which needs to be defined correctly for the sorting to
work on the input data's type. The `<list>` input should be a comma separated
list, the macros will output a comma separated list, but each element will be
wrapped in a pair of braces (so for the input `5,3,4,1` the output will be
`{1},{3},{4},{5}`), irrespective of whether you braced them in the input or not. 

### Simple Version<a name="simple"/>

The macro which needs to be correctly defined is

```latex
\is@S@ifsmaller{<new value>}{<sorted value>}{<place here>}{<try next>}
```

The macro doesn't have to be expandable. All arguments will be provided,
`<new value>` being the currently processed value, `<sorted value>` being one of
the values in the already sorted sublist, `<place here>` some code which inserts
the `<new value>` before the `<sorted value>` inside the sorted sublist, and
`<try next>` some code which will try the next element in the sorted sublist.
Both `<place here>` and `<try next>` shouldn't be inside any group or
if-else-construct.

For example to sort a list of integers, you'd use something like the following
(or an [equivalent but faster definition](#faster)):

```latex
\long\def\is@S@ifsmaller#1#2%
  {%
    \ifnum#1<#2\relax
      \expandafter\@firstoftwo
    \else
      \expandafter\@secondoftwo
    \fi
  }
```

#### Default `\is@S@ifsmaller`

The default definition of `\is@S@ifsmaller` is pretty slow, making this
version's default slower than the default of the
[complicated one](#complicated). It should work for lists which contain
integers, floats, or dimensions (even if they are mixed, using `pt` for integers
and floats), but this flexibility comes at a performance cost. If you can
guarantee the format of each element to work for a fast test you should redefine
`\is@S@ifsmaller` to that test (e.g., by using
[`\InsertionsortS`](#InsertionsortS)) and only then this version is
(considerably) faster than [its sibling](#complicated).

#### `\insertionsortS`

```latex
\insertionsortS<cs>{<list>}
```

Uses the current definition of `\is@S@ifsmaller` and defines `<cs>` to expand to
the sorted list.

#### `\InsertionsortS`<a name="InsertionsortS"/>

```latex
\InsertionsortS{<definition>}<cs>{<list>}
```

Locally uses `<definition>` for `\is@S@ifsmaller`
(`\long\def\is@S@ifsmaller#1#2{<definition>}`). *Don't* use `#3` or `#4`, just
put something like `\@firstoftwo` or `\@secondoftwo` there (maybe with
`\expandafter`). Defines `<cs>` to expand to the sorted list.

#### `\is@S@insertionsort`

```latex
\is@S@insertionsort{<list>}
```

Uses the current definition of `\is@S@ifsmaller` and defines `\is@S@sorted` to
expand to the sorted list. It does not test for an empty `<list>` argument!

### Complicated Version<a name="complicated"/>

The macro which needs to be correctly defined is

```latex
\is@C@getvalue{<element>}
```

The macro should define a macro `\is@C@value` to expand to a valid dimension, as
the comparison is done using an `\ifdim` test. The list is sorted ascending
respective of these values.

For example to sort a list of integers and/or floats, you'd use something like
the following:

```latex
\long\def\is@C@getvalue#1%
  {%
    \def\is@C@value{#1pt}%
  }
```

#### Default `\is@C@getvalue`

The default definition of `\is@C@getvalue` should work for lists containing
integers, floats, or dimensions (even if they are mixed), and sets
`\is@C@value` to a dimension (using `pt` for integers and floats). Each value
has to be calculated only once, making this version faster than the
[simple one](#simple) by default, but slower if the value is equal to the
element (or the conversion is simple, e.g., just appending `pt`) if
`\is@S@ifsmaller` is well defined. Even if you want to use this version on
simple data (though in that case the [other one](#simple) should be faster) you
can get much better performance by redefining `\is@C@getvalue` (e.g., by using
[`\InsertionsortC`](#InsertionsortC)) to reflect this.

#### `\insertionsortC`

```latex
\insertionsortC<cs>{<list>}
```

Uses the current definition of `\is@C@getvalue` and defines `<cs>` to expand to
the sorted list.

#### `\InsertionsortC`

```latex
\InsertionsortC{<definition>}<cs>{<list>}
```

Locally uses `<definition>` for `\is@C@getvalue`
(`\long\def\is@C@getvalue#1{<definition>}`).  Defines `<cs>` to expand to the
sorted list.

#### `\is@C@insertionsort`

```latex
\is@C@insertionsort{<list>}
```

Uses the current definition of `\is@C@getvalue` and defines `\is@C@sorted` to
expand to the sorted list. It does not test for an empty `<list>` argument!

### Miscellaneous

#### Faster Branching Tests<a name="faster"/>

One can define slightly faster branching if-tests without using `\expandafter`
but with the following macros. In the following examples `\if...` is replaceable
by any TeX-syntax if-test.

1. If the test is true use the first branch, else the second:
    ```latex
    \if...
      \is@fi@secondofthree
    \fi
    \@secondoftwo
    ```

1. If the test is true use the second branch, else the first:
    ```latex
    \if...
      \is@fi@thirdofthree
    \fi
    \@firstoftwo
    ```

Example: Define the integer comparison test for `\is@S@ifsmaller` from
[the simple version](#simple) with these macros:

```latex
\long\def\is@S@ifsmaller#1#2%
  {%
    \ifnum#1<#2\relax
      \is@fi@secondofthree
    \fi
    \@secondoftwo
  }
```
