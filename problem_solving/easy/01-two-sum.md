# Two Sum

**Difficulty:** Easy
**Tags:** array, hash-map

## Problem

Given a list of integers `nums` and an integer `target`, return the indices of the two numbers such that they add up to `target`. Assume exactly one valid solution exists, and you may not use the same element twice.

## Example

```
Input: nums = [2, 7, 11, 15], target = 9
Output: [0, 1]           # nums[0] + nums[1] == 9
```

```
Input: nums = [3, 2, 4], target = 6
Output: [1, 2]
```

## Constraints

- `2 <= len(nums) <= 10^4`
- `-10^9 <= nums[i] <= 10^9`
- Exactly one valid answer exists.

## Hints

1. The brute-force approach checks every pair — that's O(n²). Can you avoid the nested loop?
2. As you scan the list once, what do you need to know about numbers you've already seen?
3. Use a dictionary to map each seen value to its index; for each new number check if `target - num` is already a key.
