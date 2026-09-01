# Sliding Window Maximum

**Difficulty:** Medium
**Tags:** sliding-window, deque

## Problem

Given an array `nums` and a window size `k`, there is a sliding window of size `k` that moves from the very left to the very right of the array, one position at a time. Return the maximum value in the window at each position.

## Example

```
Input: nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3,3,5,5,6,7]
```

## Constraints

- `1 <= len(nums) <= 10^5`
- `1 <= k <= len(nums)`

## Hints

1. Recomputing the max for every window from scratch is O(n*k) — too slow for large inputs. You want each element processed roughly once.
2. Maintain a deque of *indices* such that the values at those indices are in decreasing order. The front of the deque is always the max for the current window.
3. Before adding a new index, pop from the back while its value is smaller than the new value (they can never be the max again). Also pop from the front if that index has fallen outside the current window.
