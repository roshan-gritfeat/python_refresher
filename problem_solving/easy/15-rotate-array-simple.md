# Rotate Array

**Difficulty:** Easy
**Tags:** array, list-manipulation

## Problem

Given an array, rotate it to the right by `k` steps, where `k` is non-negative. Try to do this in place.

## Example

```
Input: nums = [1,2,3,4,5,6,7], k = 3
Output: [5,6,7,1,2,3,4]
```

```
Input: nums = [-1,-100,3,99], k = 2
Output: [3,99,-1,-100]
```

## Constraints

- `1 <= len(nums) <= 10^5`
- `k >= 0` and `k` may be larger than `len(nums)`.

## Hints

1. `k` might be bigger than the array length — what should you do first to handle that? (Hint: modulo.)
2. The simplest correct approach is slicing: the last `k` elements become the new front.
3. For an O(1)-extra-space in-place trick, try reversing the whole array, then reversing the first `k` elements, then reversing the rest.
