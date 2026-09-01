# Reverse String In Place

**Difficulty:** Easy
**Tags:** string, two-pointer

## Problem

Write a function that reverses a list of characters in place (i.e. modify the input list directly, don't return a new one, and don't use `reversed()` or slicing shortcuts).

## Example

```
Input: ['h', 'e', 'l', 'l', 'o']
Output: ['o', 'l', 'l', 'e', 'h']   (list is mutated in place)
```

## Constraints

- `0 <= len(s) <= 10^5`
- `s[i]` is a printable ASCII character.

## Hints

1. Think about swapping characters from the two ends of the list moving toward the middle.
2. Use two indices, one starting at 0 and one at `len(s) - 1`.
3. Swap `s[left]` and `s[right]`, then increment `left` and decrement `right` until they meet.
