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

[`insertionsort_simple`](#simple)
: this version is faster for lists in which each element is the value, e.g., a
list of integers.

[`insertionsort_complicated`](#complicated)
: this version is faster for lists in which each element's value has to be
calculated, e.g., your list contains vectors and you want to sort them by their
lengths. It uses more tokens in a temporary macro.

For a bit more about which version is faster, see
[Speed Considerations](#speed).

You can use both versions in parallel.

## Short Documentation

Each version has a macro which needs to be defined correctly for the sorting to
work on the input data's type. The `<list>` input should be a comma separated
list, the macros will output a comma separated list, but each element will be
wrapped in a pair of braces (so for the input `5,3,4,1` the output will be
`{1},{3},{4},{5}`), irrespective of whether you braced them in the input or not.

In both versions the defaults of those two macros
([`\inso@S@ifsmaller`](#defsimp) and [`\inso@C@getvalue`](#defcomp)) take care
that you can sort lists of integers, floats, and dimensions, even if you mix
those, but the test to achieve this flexibility is optimized to be fast, not to
be rock-stable, so input which is neither of those will most likely throw errors
or in the worst case might break things and produce erroneous output.

### Simple Version<a name="simple"/>

The macro which needs to be correctly defined is

```latex
\inso@S@ifsmaller{<new value>}{<sorted value>}{<place here>}{<try next>}
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
\long\def\inso@S@ifsmaller#1#2%
  {%
    \ifnum#1<#2\relax
      \expandafter\@firstoftwo
    \else
      \expandafter\@secondoftwo
    \fi
  }
```

#### Default `\inso@S@ifsmaller`<a name="defsimp"/>

The default definition of `\inso@S@ifsmaller` is pretty slow, making this
version's default slower than the default of the
[complicated one](#complicated). It should work for lists which contain
integers, floats, or dimensions (even if they are mixed, using multiples of the
dimen `\inso@unit` for integers and floats), but this flexibility comes at a
performance cost. If you can guarantee the format of each element to work for a
fast test you should redefine `\inso@S@ifsmaller` to that test (e.g., by using
[`\InsertionsortS`](#InsertionsortS)) and only then this version is
(considerably) faster than [its sibling](#complicated).

#### `\insertionsortS`

```latex
\insertionsortS<cs>{<list>}
```

Uses the current definition of `\inso@S@ifsmaller` and defines `<cs>` to expand
to the sorted list.

#### `\InsertionsortS`<a name="InsertionsortS"/>

```latex
\InsertionsortS{<definition>}<cs>{<list>}
```

Locally uses `<definition>` for `\inso@S@ifsmaller`
(`\long\def\inso@S@ifsmaller#1#2{<definition>}`). *Don't* use `#3` or `#4`, just
put something like `\@firstoftwo` or `\@secondoftwo` there (maybe with
`\expandafter`). Defines `<cs>` to expand to the sorted list.

#### `\inso@S@insertionsort`

```latex
\inso@S@insertionsort{<list>}
```

Uses the current definition of `\inso@S@ifsmaller` and defines `\inso@S@sorted`
to expand to the sorted list. It does not test for an empty `<list>` argument!

### Complicated Version<a name="complicated"/>

The macro which needs to be correctly defined is

```latex
\inso@C@getvalue{<element>}
```

The macro should define a macro `\inso@C@value` to expand to a valid dimension,
as the comparison is done using an `\ifdim` test. The list is sorted ascending
respective of these values.

For example to sort a list of integers and/or floats, you'd use something like
the following:

```latex
\long\def\inso@C@getvalue#1%
  {%
    \def\inso@C@value{#1pt}%
  }
```

#### Default `\inso@C@getvalue`<a name="defcomp"/>

The default definition of `\inso@C@getvalue` should work for lists containing
integers, floats, or dimensions (even if they are mixed), and sets
`\inso@C@value` to a dimension (using multiples of the dimen `\inso@unit` for
integers and floats). Each value has to be calculated only once, making this
version faster than the [simple one](#simple) by default, but slower if the
value is equal to the element (or the conversion is simple, e.g., just appending
`pt`) if `\inso@S@ifsmaller` is well defined. Even if you want to use this
version on simple data (though in that case the [other one](#simple) should be
faster) you can get much better performance by redefining `\inso@C@getvalue`
(e.g., by using [`\InsertionsortC`](#InsertionsortC)) to reflect this.

#### `\insertionsortC`

```latex
\insertionsortC<cs>{<list>}
```

Uses the current definition of `\inso@C@getvalue` and defines `<cs>` to expand
to the sorted list.

#### `\InsertionsortC`

```latex
\InsertionsortC{<definition>}<cs>{<list>}
```

Locally uses `<definition>` for `\inso@C@getvalue`
(`\long\def\inso@C@getvalue#1{<definition>}`).  Defines `<cs>` to expand to the
sorted list.

#### `\inso@C@insertionsort`

```latex
\inso@C@insertionsort{<list>}
```

Uses the current definition of `\inso@C@getvalue` and defines `\inso@C@sorted`
to expand to the sorted list. It does not test for an empty `<list>` argument!

### Miscellaneous

The dimen `\inso@unit` is used to convert integers and floats into dimensions in
the defaults of `\inso@C@getvalue` and `\inso@S@ifsmaller`. You might change its
value (e.g., `\inso@unit=2pt`), but don't redefine it, it is important that
`\inso@unit` is a dimen. The default is `\inso@unit=1pt`.

#### Faster Branching Tests<a name="faster"/>

One can define slightly faster branching if-tests without using `\expandafter`
but with the following macros. In the following examples `\if...` is replaceable
by any TeX-syntax if-test. For the `\inso@fi@...` macros the next tokens must be
`\fi` and the corresponding `\inso@...` macro.

1. If the test is true use the first branch, else the second:
    ```latex
    \if...
      \inso@fi@firstoftwo
    \fi
    \inso@secondoftwo
    ```

1. If the test is true use the second branch, else the first:
    ```latex
    \if...
      \inso@fi@secondoftwo
    \fi
    \inso@firstoftwo
    ```

1. If the test is true use the next brach, else gobble it:
    ```latex
    \if...
      \inso@fi@firstofone
    \fi
    \inso@gobble
    ```

1. If the test is true gobble the next brach, else use it:
    ```latex
    \if...
      \inso@fi@gobble
    \fi
    \inso@firstofone
    ```

Example: Define the integer comparison test for `\inso@S@ifsmaller` from
[the simple version](#simple) with these macros:

```latex
\long\def\inso@S@ifsmaller#1#2%
  {%
    \ifnum#1<#2\relax
      \inso@fi@firstoftwo
    \fi
    \inso@secondoftwo
  }
```

#### Speed Considerations<a name="speed"/>

In the descriptions of the defaults of [`\inso@S@ifsmaller`](#defsimp) and
[`\inso@C@getvalue`](#defcomp) I gave a bit of information about which version
should be faster. The truth is, since the [complicated version](#complicated)
roughly doubles the tokens read and saved during the sorting, it gets slower for
large lists and it is slower for really small lists. For random integer arrays
but with the defaults of both versions the [simple version](#simple) is faster
for arrays of 5 or fewer elements. In the range 10 to 80 elements the
[complicated version](#complicated) is faster, but somewhere between 80 and 160
elements the [simple one](#simple) overtakes again.

Generally the algorithm has a runtime of $O(n^2)$, so the performance gets
significantly worse for big arrays, also, since the way the data is handled
(saving in a macro, having TeX reread it each iteration in the process),
this implementation has even worse performance.

But for small lists the performance is pretty good, outperforming the sort
algorithm of `lambda.sty` (as long as the input is not an already sorted list,
and having the drawback of not being expandable)
and even performing comparable to the mergesort of `l3sort`. To be more precise
for random integer input this implementation should be faster than `l3sort` for
up to 40 elements on average. To be fair, I should note that I compared
`l3sort`'s performance with `\clist_set:Nn` and `\clist_sort:Nn`, which also
sanitizes the input and removes leading and trailing spaces. If both can assume
well-stated input (so don't have to clean the input, in the case of `l3sort` not
using `\clist_set:Nn`), still being faster for up to 20 elements, with `l3sort`
becoming faster somewhere between 20 and 40, and still being in the same
order of magnitude for 40 elements.
