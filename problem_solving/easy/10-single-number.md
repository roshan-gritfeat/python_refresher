# Single Number

**Difficulty:** Easy
**Tags:** array, bit-manipulation

## Problem

Given a non-empty array of integers where every element appears exactly twice except for one element which appears exactly once, find that single element. Try to do it in linear time with constant extra space.

## Example

```
Input: nums = [4, 1, 2, 1, 2]
Output: 4

Input: nums = [2, 2, 1]
Output: 1
```

## Constraints

- `1 <= len(nums) <= 3 * 10^4`
- Every element appears exactly twice except for one.

## Hints

1. A hash-set based "seen it before, remove it" approach works but uses O(n) extra space — can you do O(1) space?
2. Recall the XOR properties: `x ^ x == 0` and `x ^ 0 == x`, and XOR is commutative/associative.
3. XOR all the numbers together in a single pass — every paired value cancels out, leaving only the single number.
