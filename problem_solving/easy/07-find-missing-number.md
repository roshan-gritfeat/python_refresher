# Find Missing Number

**Difficulty:** Easy
**Tags:** math, array

## Problem

Given an array `nums` containing `n` distinct numbers taken from the range `[0, n]`, find the one number in the range that is missing from the array.

## Example

```
Input: nums = [3, 0, 1]
Output: 2

Input: nums = [9, 6, 4, 2, 3, 5, 7, 0, 1]
Output: 8
```

## Constraints

- `n == len(nums)`
- `1 <= n <= 10^4`
- All numbers are unique.

## Hints

1. You could sort and scan for a gap, but is there an approach that avoids sorting?
2. Think about the sum of `0..n` versus the actual sum of the array.
3. `expected_sum - actual_sum` gives you the missing number directly, in O(n) time and O(1) space. (Bonus: try it with XOR instead of sum.)
