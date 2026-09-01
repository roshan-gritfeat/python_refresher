# Subsets

**Difficulty:** Medium
**Tags:** backtracking, recursion

## Problem

Given an array `nums` of unique integers, return all possible subsets (the power set). The solution set must not contain duplicate subsets.

## Example

```
Input: nums = [1,2,3]
Output: [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]   # order may vary
```

## Constraints

- `1 <= len(nums) <= 10`
- All elements are unique.

## Hints

1. For each element, there are exactly two choices: include it in the current subset, or don't. That branching is the essence of backtracking.
2. Write a recursive function that takes an index and a "current subset so far"; at each index, recurse both with and without including `nums[index]`.
3. Alternatively, build subsets iteratively: start with `[[]]` and for each number, add it to every existing subset to double the list of subsets.
