# Kth Largest Element in an Array

**Difficulty:** Medium
**Tags:** heap

## Problem

Given an integer array `nums` and an integer `k`, return the k-th largest element in the array (the k-th largest in sorted order, not the k-th distinct element).

## Example

```
Input: nums = [3,2,1,5,6,4], k = 2
Output: 5

Input: nums = [3,2,3,1,2,4,5,5,6], k = 4
Output: 4
```

## Constraints

- `1 <= k <= len(nums) <= 10^5`

## Hints

1. Sorting the whole array and indexing works but is O(n log n) — can you do better when k is small relative to n?
2. A min-heap of size `k` lets you track the k largest elements seen so far efficiently; the heap's smallest element (the top) is the answer once you've processed everything.
3. Python's `heapq` is a min-heap by default. Push elements, and whenever the heap grows beyond size `k`, pop the smallest. At the end, `heap[0]` is your answer. (Alternative: look into `heapq.nlargest` or the quickselect algorithm for O(n) average time.)
