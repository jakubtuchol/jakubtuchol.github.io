---
layout: post
title:  "Interview Questions: Finding the smallest missing integer"
date:   2019-02-28 16:00:00
categories: interview-questions
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
    let max_num = nums.len();
    for num in nums {
        if *num > 0 {
            seen.insert(*num);
        }
    }

    for i in 1..max_num + 1 {
        if !seen.contains(&(i as i32)) {
            return i as i32;
        }
    }
    (max_num + 1) as i32
}
```

### Performance

This solution provides the afore-mentioned time-space tradeoff. It utilizes `O(n)` space, since in the worst case, all the elements can be unique positive integers and therefore indexed in the tracking set. As expected, the runtime improvides significantly, providing us with `O(n)` runtime, since you have to iterate over number of elements directly proportional to the size of the array multiple times.

## Marking Solution

Given that our input is unordered, and given that the problem implies the need to examine every element within an array to find the missing element, we can't do much better than `O(n)` runtime. We can actually improve on the space utilization aspect of the previous solution if we make one simple assumption: the array provided is mutable and we can modify the elements of the array in-place. Given this assumption, we can leverage the array itself as a proxy for the set of seen positive integers that we utilized in the previous problem.

The idea is relatively simple: if there is a missing integer, we know that the smallest missing integer must be less than or equal to the length of the array. Given this, as we iterate through the array to get a sense of the positive integers that are present, we can denote the presence of a positive integer by *marking* the space at the corresponding index. If we limit ourselves to only operating over the set of positive numbers with in the array, we can perform this marking by simply setting the element at the index of interest to be negative. We can then run through the array to find the first positive number and return this index as the first missing positive number. Let's run through an example using an array of only positive numbers.

1. We start with the following list:
  * `[2, 3, 6, 8, 1, 15]`
2. We begin iterating through the numbers of the array. We first see the number `2`, which means that we're going to mark the number at position `2` to be negative. Since positive numbers begin at 1, and our array can (in the "worst" case) have `n` positive numbers, we need to adjust the associated indices to begin indexing at 1. Hence, we'll mark the number at position `1` as present.
  * `[2, -3, 6, 2, 8, 1, 15]`
3. Next we find `-3`. Since we know that all numbers within this array must be positive, we have to take the absolute value of `-3` (aka `3`) to be the original value. We then mark the element at index `2` to be negative:
  * `[2, -3, -6, 2, 8, 1, 15]`
4. We now do the same thing as before with `-6` to and mark the element at index `5` as negative:
  * `[2, -3, -6, 2, 8, -1, 15]`
5. We now find another `2`. Since the element at index `1` is already negative, we leave it as is:
  * `[2, -3, -6, 2, 8, -1, 15]`
6. We now encounter `8`. Since `8` is larger than the size of the array, we don't do anything, since our set of consecutive positive numbers cannot reach 8:
  * `[2, -3, -6, 2, 8, -1, 15]`
7. We now do the same thing for the remaining numbers in the array:
  * `[-2, -3, -6, 2, 8, -1, 15]`
8. Given that all present elements have been marked in the array, we can now iterate through the array to find the first missing number. That number occurs at index `3`, which corresponds to the number `4`. Hence, we return `4` as the answer.

Now, this approach does not work so well when we already have negative numbers present in the array. We can fix this by segregating all the non-positive numbers to the left side of the array, and then restricting ourselves to operate over the right side of the array (which contains only positive numbers). For example, if we were given the array `[2, 3, -7, 6, 8, 1, -10, 15]`, we would first segregate the elements to give us an array like: `[-10, -7, 2, 3, 6, 8, 1, 15]`. We would then operate over the right side of our array to get a solution.

The code for this solution is provided below:

```rust
fn find_missing_number(nums: &mut [i32]) -> i32 {
    fn segregate(nums: &mut [i32]) -> usize {
        let mut num_negative: usize = 0;
        let mut idx = 0;

        while idx < nums.len() {
            if nums[idx] <= 0 {
                let temp = nums[idx];
                nums[idx] = nums[num_negative];
                nums[num_negative] = temp;
                num_negative += 1;
            }
            idx += 1;
        }
        num_negative
    }

    let num_negative = segregate(nums);
    let max_idx = nums.len();
    let num_positive = max_idx - num_negative;

    let mut idx = num_negative;
    while idx < max_idx {
        let marked_idx = (nums[idx].abs() - 1) as usize;
        if marked_idx < num_positive {
            nums[marked_idx + num_negative] = nums[marked_idx + num_negative].abs() * -1;
        }
        idx += 1;
    }

    idx = num_negative;
    while idx < max_idx {
        if nums[idx] > 0 {
            return (idx - num_negative + 1) as i32;
        }
        idx += 1;
    }

    (num_positive + 1) as i32
}
```

### Performance

Given the fact that we're modifying the array in-place and using no additional space, we're using `O(1)` space. All of our iterations are linear iterations through the array, which means that our algorithm runs in `O(n)` time.

There are a couple of other solutions to this problem, but performance-wise, we really can't do much better than what we have here. The core take-away here is realizing that we can push our space utilization down drastically by making use of the array we already have available to us.
