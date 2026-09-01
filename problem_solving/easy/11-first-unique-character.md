# First Unique Character in a String

**Difficulty:** Easy
**Tags:** string, hash-map

## Problem

Given a string `s`, find the index of the first non-repeating character in it and return that index. If it does not exist, return `-1`.

## Example

```
Input: s = "leetcode"
Output: 0        # 'l' never repeats

Input: s = "loveleetcode"
Output: 2        # 'v' is the first char that appears only once

Input: s = "aabb"
Output: -1
```

## Constraints

- `1 <= len(s) <= 10^5`
- `s` consists of lowercase English letters.

## Hints

1. You need to know how many times each character appears in the whole string first.
2. Build a frequency count with `collections.Counter`, then make a second pass over the string in order.
3. Return the index of the first character in the second pass whose count is exactly 1.
