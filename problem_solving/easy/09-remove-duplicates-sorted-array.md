# Remove Duplicates from Sorted Array

**Difficulty:** Easy
**Tags:** array, two-pointer

## Problem

Given a sorted array `nums`, remove the duplicates in place such that each unique element appears only once, and return the number of unique elements. The relative order of elements should be kept, and the first `k` elements of `nums` should hold the final result (you don't need to worry about elements beyond that).

## Example

```
Input: nums = [1, 1, 2]
Output: k = 2, nums = [1, 2, _]
```

```
Input: nums = [0,0,1,1,1,2,2,3,3,4]
Output: k = 5, nums = [0,1,2,3,4,_,_,_,_,_]
```

## Constraints

- `1 <= len(nums) <= 3 * 10^4`
- `nums` is sorted in non-decreasing order.

## Hints

1. Because the array is sorted, all duplicate values are adjacent — you don't need a set to track what you've seen.
2. Use two pointers: a "slow" pointer marking where the next unique value should be written, and a "fast" pointer scanning ahead.
3. Whenever `nums[fast] != nums[slow]`, increment `slow` and copy `nums[fast]` there.
