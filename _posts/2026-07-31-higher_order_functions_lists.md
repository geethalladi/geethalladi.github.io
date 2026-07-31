---
title: "Higher Order Functions in Python - Part I"
date: 2026-07-31
layout: post
description: "Small idioms that make code read like prose"
categories:
- python
tags:
- python
- functional-programming
- higher-order-functions
- list-comprehensions
---

# Table of Contents

1.  [Introduction](#org7ea5387)
2.  [Why higher order functions](#org505a743)
3.  [Asking questions about a whole list](#org3558ff6)
4.  [List Comprehensions: map and filter with nicer syntax](#org98cf257)
5.  [Reduce — collapsing a list to one value](#orgc1c5d2b)
6.  [The walrus operator in list comprehensions](#orgee646e8)
7.  [Wrapping up](#org47e49b7)

<a id="org7ea5387"></a>

# Introduction

Most application functions touches a list that is doing one of the following three things

-   Ask a question about every element
-   Transforming every element
-   Collapsing the elements into a single value

Once you can name those three shapes - *predicate*, *map* and *reduce* a surprising amount of loop-and-accumulator code collapses into a single readable line.

This first part sticks to the idioms Python gives you out of the box for lists: `all`, `any`, `max`, `min`, `comprehensions`, `reduce`, and the `walrus operator`. The subsequent parts will talk about `dictionary`, `functools`, `itertools`, and writing your own higher order functions.


<a id="org505a743"></a>

# Why higher order functions

A **higher order function** is simply a function that takes another function as an argument, returns a function, or both. `map`, `filter`, `sorted` and
`reduce` are all higher order functions. The following are the advantages of leaning on them

-   **Concise**: intent fits on one line instead of five.
-   **Readable**: `all(...)` says "every one of these holds" far more directly than a loop with an early `return False`.
-   **Easier to debug**: known idioms have known failure modes. A reader (and future you) can skip past `max(xs)` without checking it for off-by-one bugs.

**- Immutable-friendly**: expressions produce new values instead of mutating an accumulator in place, so there is no half-updated intermediate state to
  reason about. This in turn favors concurrent execution

-   **Unix-flavoured composition**: write small functions that each do one thing well, then wire them together to get the behaviour the application actually
    needs. The pipeline, not the function, is where the application lives.

That last point is the real one. The idioms below are the pipe operators of Python data code.


<a id="org3558ff6"></a>

# Asking questions about a whole list


<a id="orgd4474cf"></a>

## `all` - does every element hold?

`all` returns `True` when every element is truthy

```python
    all([True, 1, {}])   # {} is falsy
    # => False

    all([True, 1, {0}])  # a non-empty set is truthy
    # => True
```


<a id="org46822a8"></a>

## `any` - does at least one element hold?

`any` returns `True` when any one element is truthy

```python
    any([True, 1, {}])
    # => True

    any([])
    # => False
```

Both functions short-circuit

```python
    # stops at the first negative number; never scans the rest
    any(x < 0 for x in [1, 2, -3, 4, 5])
    # => True
```


<a id="orgc21f5ea"></a>

## `max` and `min` - the extremes in a list

```python
    xs = [3, 2, 4, 1, 5]

    max(xs)
    # => 5

    min(xs)
    # => 1

    # over a slice
    max(xs[2:4])
    # => 4

    max([], default=None)
    # => None

    people_names = ['alice', 'bob', 'charlie']
    max(people_names, key=len)
    # => 'charlie'
```


<a id="org98cf257"></a>

# List Comprehensions: map and filter with nicer syntax

Python has `map` and `filter` builtins, but comprehensions are the idiomatic spelling of both. Some sample data to work with:

```python
    from dataclasses import dataclass

    @dataclass
    class Person:
        name: str
        dept: str

    people = [
        Person('alice', 'engg'),
        Person('bob', 'sales'),
        Person('charlie', 'HR'),
    ]
```

<a id="org39857a6"></a>

## Map - transform every element


### Convert People to their names

```python
    # [Person] -> [name]
    names = [p.name for p in people]
    # => ['alice', 'bob', 'charlie']
```

### Bucketing scores into decades

```python
    def bucket(grade):
        """Round a grade down to its decade: 79 -> 70."""
        return grade // 10 * 10

    grades = [79, 25, 31, 72, 60, 10, 65, 32, 100, 63, 36, 57, 0]

    [bucket(g) for g in grades]
    # => [70, 20, 30, 70, 60, 10, 60, 30, 100, 60, 30, 50, 0]
```


<a id="org6548197"></a>

## Ignoring the element entirely

```python
    zeros = [0 for _ in [1, 2, 3, 4]]
    # => [0, 0, 0, 0]
```

<a id="org19a5467"></a>

## Nested loops - building pairs

```python
    increasing_pairs = [(x, y)
                        for x in range(4)             # outer loop
                        for y in range(x + 1, 4)]     # depends on x, so y > x
    # => [(0, 1), (0, 2), (0, 3), (1, 2), (1, 3), (2, 3)]
```

Because `range(lo, hi)` excludes `hi`, starting the inner range at `x + 1` is what guarantees $x < y$ without a filter.


<a id="org469ba7b"></a>

## Filter - keep only what matches

```python
    hrs = [p.name for p in people if p.dept == "HR"]
    # => ['charlie']

    even_nums = [x for x in range(5) if x % 2 == 0]
    # => [0, 2, 4]
```

<a id="orgc1c5d2b"></a>

# Reduce — collapsing a list to one value

`reduce` threads a two-argument function through the list, carrying an accumulator along

```python
    from functools import reduce

    total = reduce(lambda acc, y: acc + y, [1, 2, 3, 4, 5])
    # => 15
```

An optional third argument is the starting accumulator, which is also what `reduce` returns for an empty list

```python
    reduce(lambda acc, p: acc + ", " + p.name, people, "")
    # => ', alice, bob, charlie'

    ", ".join(p.name for p in people)
    # => 'alice, bob, charlie'
```

<a id="org0dc604c"></a>

## Rule of thumb

If a builtin already names your fold like `sum`, `min`, `max`, `any`, `all`, `str.join` then use it. Save `reduce` for genuinely custom accumulations, and consider a plain loop when the reducing function stops fitting on one line.


<a id="orgee646e8"></a>

# The walrus operator in list comprehensions

A filter often needs the same computed value that the output expression needs. The naive version computes it twice

```python
    def inc(x):
        return x + 1

    xs = [-5, -2, -1, 6, 7, 9, -3, -4, 3]

    [inc(x) for x in xs if inc(x) > 0]
    # => [7, 8, 10, 4]
```

Two calls to `inc` per element. Harmless here; not harmless when `inc` is a database lookup or a regex match. The assignment expression `:=` binds the result once and reuses it

```python
    [y for x in xs if (y := inc(x)) > 0]
    # => [7, 8, 10, 4]
```

This pattern shines with `re.match`

```python
    import re

    lines = ["id=10", "junk", "id=42"]

    [int(m.group(1)) for line in lines if (m := re.match(r"id=(\d+)", line))]
    # => [10, 42]
```


<a id="org47e49b7"></a>

# Wrapping up

The through-line of every idiom above is the same: name the shape of the computation instead of re-deriving it with an index and an accumulator.

-   `all` / `any` for questions over a whole collection, short-circuiting.
-   `max` / `min` with `key` and `default` for extremes.
-   Comprehensions for map and filter, including nested and dependent loops.
-   `reduce` for folds that no builtin already covers.
-   The walrus operator to avoid computing the same thing twice.

In the subsequent parts we'll go a level up: `hash maps`, `functools.partial` and `lru_cache`, the `itertools` toolkit, `sorted` with composite keys, decorators as function-returning functions, and how to build small pipelines out of functions you wrote yourself.
