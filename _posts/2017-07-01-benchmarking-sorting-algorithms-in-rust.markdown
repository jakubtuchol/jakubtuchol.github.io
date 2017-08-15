---
layout: post
title:  "Benchmarking Sorting Algorithms in Rust"
date:   2017-07-01 09:00:00
categories: algorithms rust
---

Every algorithms textbook includes a robust discussion of sorting algorithms, with the associated big-O runtimes. While it's very easy to be able to point to these asymptotic runtime analyses as the final word on the performance of these sorting algorithms, I was very interested in determining whether or not these theoretical analyses actually described real runtime performance.

# The Algorithms

We will be working with six classic sorting algorithms:

* insertion sort
* quick sort
* merge sort
* heapsort
* bubble sort
* shell sort

# Runtime Characteristics

| algorithm      | best      | average   | worst     | memory  |
|----------------|-----------|-----------|-----------|---------|
| quicksort      | `n log n` | `n log n` | `n^2`     | `log n` |
| merge sort     | `n log n` | `n log n` | `n log n` | `n`     |
| heapsort       | `n log n` | `n log n` | `n log n` | 1       |
| insertion sort | `n`       | `n^2`     | `n^2`     | 1       |
| shell sort     | `n log n` | `n log n` | `n log n` | 1       |
| bubble sort    | `n`       | `n log n` | `n log n` | 1       |

# Benchmarking

# Implementations

## Insertion Sort
