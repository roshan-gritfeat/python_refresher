# Longest Substring Without Repeating Characters

**Difficulty:** Medium
**Tags:** sliding-window, hashing

## Problem

Given a string `s`, find the length of the longest substring without repeating characters.

## Example

```
Input: s = "abcabcbb"
Output: 3        # "abc"

Input: s = "bbbbb"
Output: 1        # "b"

Input: s = "pwwkew"
Output: 3        # "wke"
```

## Constraints

- `0 <= len(s) <= 5 * 10^4`
- `s` consists of English letters, digits, symbols, and spaces.

## Hints

1. The brute-force approach checks every substring — O(n²) or worse. Think about maintaining a "window" instead.
2. Use two pointers (`left`, `right`) defining a window, and a set/dict tracking characters currently inside it. Expand `right`; when you hit a duplicate, shrink from `left`.
3. Storing the last-seen index of each character in a dict lets you jump `left` directly past the duplicate instead of shrinking one step at a time.
