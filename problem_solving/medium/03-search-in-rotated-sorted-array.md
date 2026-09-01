# Search in Rotated Sorted Array

**Difficulty:** Medium
**Tags:** binary-search

## Problem

You are given an array of distinct integers `nums` sorted in ascending order, but then rotated at some unknown pivot (e.g. `[0,1,2,4,5,6,7]` might become `[4,5,6,7,0,1,2]`). Given a `target` value, return its index in `nums`, or `-1` if it's not present. Your algorithm must run in O(log n) time.

## Example

```
Input: nums = [4,5,6,7,0,1,2], target = 0
Output: 4

Input: nums = [4,5,6,7,0,1,2], target = 3
Output: -1
```

## Constraints

- `1 <= len(nums) <= 5000`
- All values in `nums` are unique.
- `nums` is guaranteed to be rotated at some pivot.

## Hints

1. A plain linear scan is O(n) — think about how binary search can still work even though the array isn't fully sorted.
2. At every step of binary search, at least one half of the current range (left-to-mid or mid-to-right) IS normally sorted. Figure out which half is sorted by comparing `nums[left]`, `nums[mid]`, `nums[right]`.
3. Once you know which half is sorted, check if `target` falls within that half's range to decide which half to recurse/iterate into.
