# Merge Intervals

**Difficulty:** Medium
**Tags:** sorting, intervals

## Problem

Given a list of intervals where `intervals[i] = [start_i, end_i]`, merge all overlapping intervals and return an array of the non-overlapping intervals that cover all the intervals in the input.

## Example

```
Input: intervals = [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]

Input: intervals = [[1,4],[4,5]]
Output: [[1,5]]
```

## Constraints

- `1 <= len(intervals) <= 10^4`
- `intervals[i].length == 2`
- `0 <= start_i <= end_i`

## Hints

1. Overlaps are much easier to detect once the intervals are in a predictable order.
2. Sort the intervals by their start value first.
3. Walk through the sorted intervals keeping a "current merged interval"; if the next interval's start is `<=` the current merged interval's end, extend the end (`max` of the two ends); otherwise, close off the current merged interval and start a new one.
