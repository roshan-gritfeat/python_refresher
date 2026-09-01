# 3Sum

**Difficulty:** Medium
**Tags:** two-pointer, sorting

## Problem

Given an array of integers `nums`, return all unique triplets `[a, b, c]` such that `a + b + c == 0`. The solution set must not contain duplicate triplets.

## Example

```
Input: nums = [-1,0,1,2,-1,-4]
Output: [[-1,-1,2],[-1,0,1]]
```

## Constraints

- `3 <= len(nums) <= 3000`
- `-10^5 <= nums[i] <= 10^5`

## Hints

1. Brute force is O(n³) with three nested loops. Sorting the array first opens up a faster approach.
2. Fix one element `nums[i]`, then use the two-pointer technique on the remaining sorted subarray to find pairs that sum to `-nums[i]` — this is exactly the "two sum on a sorted array" pattern.
3. To avoid duplicate triplets, skip over repeated values for `i`, and after finding a valid pair, skip repeated values for both the left and right pointers before continuing.
