---
title: "Binary Search and Python's bisect module"
date: 2026-05-31
layout: post
categories:
- algorithms
tags:
- binary_search
- retrieval
- sorted
- python
- algorithms
---

# Table of Contents

1.  [Binary Search & Python's bisect module](#org2baacdb)
    1.  [Introduction](#org3a00e47)
    2.  [Why binary search?](#orgafaafd7)
    3.  [Naive implementation](#org02a3586)
    4.  [bisect\_left — leftmost occurrence](#org3218183)
    5.  [bisect\_right — rightmost occurrence](#org549604b)
    6.  [When to use which](#org3c67372)



<a id="org2baacdb"></a>

# Binary Search & Python's bisect module


<a id="org3a00e47"></a>

## Introduction

Understanding the thought process behind `bisect_left` and `bisect_right` — when to use each and why.


<a id="orgafaafd7"></a>

## Why binary search?

Linear search scans every element in O(n). Binary search exploits sorted order to reduce the run time to O(log n). That's the difference between scanning a million records and needing at most 20 comparisons. It's also the idea powering B-tree indexes in most databases.

Use binary search whenever your data is sorted and you need fast lookups, range queries, or insertion-point finding.


<a id="org02a3586"></a>

## Naive implementation

The following is a classic "book" implementation of the algorithm which returns an index if found, `-1` otherwise.

    def binary_search(xs, key):
        if not xs or key is None:
            return -1
        left, right = 0, len(xs) - 1
        while left <= right:
            mid = (left + right) // 2
            if xs[mid] == key:
                return mid
            elif xs[mid] < key:
                left = mid + 1
            else:
                right = mid - 1
        return -1

    xs = [1, 2, 3, 4, 5]
    assert binary_search(xs, 7) == -1
    assert binary_search(xs, 5) == 4
    assert binary_search(xs, 1) == 0

This returns *some* index when a match is found. That's fine for unique elements, but falls apart with duplicates. Enter
[bisect module](https://docs.python.org/3/library/bisect.html) included in the [Python Standard library](https://docs.python.org/3/library/index.html).


<a id="org3218183"></a>

## bisect\_left — leftmost occurrence


### Invariants

-   Returns position `i` such that `a[:i] < key` and `a[i:] >=` key
-   If key is present, `i` points to its **leftmost** copy
-   Return range: `[0, len(a)]` — can return an index past the end


### Implementation

    from bisect import bisect_left

    def left_most_search(a, key):
        i = bisect_left(a, key)
        if i >= len(a) or a[i] != key:  # bounds check first!
            return -1
        return i

    xs = [1, 2, 2, 2, 3, 4, 5]
    assert left_most_search(xs, 2) == 1
    assert left_most_search(xs, 3) == 4
    assert left_most_search(xs, 9) == -1


<a id="org549604b"></a>

## bisect\_right — rightmost occurrence


### Invariants

-   Returns position `i` such that `a[:i] <=` key and `a[i:] > key`
-   Never points to the key itself — always one position *after* it
-   Return range: `[0, len(a)]`


### Implementation

    from bisect import bisect_right

    def right_most_search(a, key):
        i = bisect_right(a, key)
        if i == 0 or a[i - 1] != key:
            return -1
        return i - 1

    xs = [1, 2, 2, 2, 3, 4, 5]
    assert right_most_search(xs, 2) == 3
    assert right_most_search(xs, 1) == 0
    assert right_most_search(xs, 9) == -1


<a id="org3c67372"></a>

## When to use which


### bisect\_left

-   Finding the **first** occurrence of a value
-   Insertion point that preserves order, placing equal keys to the left
-   Counting elements strictly less than a value: `bisect_left(a, key)` gives the count


### bisect\_right

-   Finding the **last** occurrence of a value
-   Insertion point that places equal keys to the right
-   Counting elements less than or equal to a value: `bisect_right(a, key)` gives the count
