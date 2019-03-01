---
layout: post
title:  "Finding the smallest missing integer"
date:   2019-02-28 16:00:00
categories: interview-questions rust
---

In this post, we'll be covering a common interview problem and a series of solutions for it.

## The Problem

Given an array of integers, find the first missing positive integer. To be explicit, find the lowest positive integer that does not exist in the array. The array can contain duplicates and negative numbers as well.

### Example

* *Input*: `[1, -1, 0, -1, -2, 3]`
* *Output*: 2

## Sorting Solution

A very simple way to handle this is to take the array, and sort the array. Once the array is sorted, we can then iterate through the array and find the first missing integer by keeping a counter of the last seen positive number and checking if the next largest positive number is one more than the last number. If it is not, then we returns the next expected number. As an example, we can take the following array:

```
[1, -1, 0, -1, -2, 3]
```

We first remove the non-positive numbers and sort the result:

```
[1, 3]
```

We then set the initial counter to `0` and begin iterating. Since the next largest number is `1` more than `0`, we set the counter to `1` and continue iterating. The next number `3`, is not `1` more than `1`, so we return the next expected number, which is `2`.

If we find that all numbers are present, then we simply return the next expected integer. The code for this solution is shown below:

```rust
fn find_missing_number(nums: &mut [i32]) -> i32 {
    nums.sort();

    let mut last_positive = 0;
    for elt in nums {
        if elt > &mut last_positive {
            if elt == &mut (last_positive + 1) {
                last_positive += 1;
            } else {
                return last_positive + 1;
            }
        }
    }
    return last_positive + 1;
}
```

### Performance

This algorithm uses no additional space, so the space utilization is `O(1)`. The majority of this algorithm runs in linear time, since it performs a single scan across the sorted array, but the majority of the runtime comes from sorting the array, which runs in `O(n log n)` time.

## Set Solution

So the previous solution was relatively clean, but it seems that we could do a bit better from a performance perspective. Generally, when we're trying to improve algorithms, we can get performance optimizations by allowing our space utilization to creep up. In this case, we can use a constant-time lookup data structure, such as a set, to allow for us to keep track of what numbers we've seen. Concretely, we can iterate through our array, and add any positive integers to a set of numbers that we have seen. Once we have iterated through our array, we can then iterate from 1 to the size of the array, and return the first number not found in the set of numbers that we have seen. If we have seen all these numbers, we can return the size of the array plus one.

Concretely, the solution would look like this:

```rust
fn find_missing_number(nums: &[i32]) -> i32 {
    let mut seen = HashSet::new();
    for num in nums {
        if *num > 0 {
            seen.insert(*num);
        }
    }

    for i in 1..nums.len() - 1 {
        if !seen.contains(&(i as i32)) {
            return i as i32;
        }
    }
    return nums.len() as i32;
}
```

### Performance

This solution provides the afore-mentioned time-space tradeoff. It utilizes `O(n)` space, since in the worst case, all the elements can be unique positive integers and therefore indexed in the tracking set. As expected, the runtime improvides significantly, providing us with `O(n)` runtime, since you have to iterate over number of elements directly proportional to the size of the array multiple times.

## Marking Solution
