# Valid Anagram

**Difficulty:** Easy
**Tags:** string, hash-map, counting

## Problem

Given two strings `s` and `t`, return `True` if `t` is an anagram of `s` (i.e. uses exactly the same letters with the same frequency, possibly in a different order), and `False` otherwise.

## Example

```
Input: s = "anagram", t = "nagaram"
Output: True

Input: s = "rat", t = "car"
Output: False
```

## Constraints

- `1 <= len(s), len(t) <= 5 * 10^4`
- Both strings consist of lowercase English letters.

## Hints

1. If the two strings have different lengths, they can't be anagrams — check that first as a quick exit.
2. Counting character frequencies is the key idea. `collections.Counter` does this in one line.
3. Two strings are anagrams exactly when `Counter(s) == Counter(t)`.
