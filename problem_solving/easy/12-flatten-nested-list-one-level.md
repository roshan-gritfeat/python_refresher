# Flatten a Nested List (One Level Deep)

**Difficulty:** Easy
**Tags:** list, recursion

## Problem

Given a list that may contain both integers and sub-lists (nested exactly one level deep — sub-lists themselves contain only integers, not further lists), return a single flat list of integers preserving order.

## Example

```
Input: [1, [2, 3], 4, [5, 6, 7]]
Output: [1, 2, 3, 4, 5, 6, 7]
```

```
Input: [[1], [], [2, 3]]
Output: [1, 2, 3]
```

## Constraints

- `0 <= len(nested_list) <= 1000`
- Nesting depth is exactly 0 or 1 (no deeper nesting for this version).

## Hints

1. Iterate through the top-level list and check the type of each item.
2. If an item is a list, extend your result with its contents; otherwise append the item directly.
3. `isinstance(item, list)` is the check you want; a one-line solution is possible with a nested list comprehension.
