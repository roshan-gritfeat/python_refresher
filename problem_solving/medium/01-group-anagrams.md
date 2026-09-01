# Group Anagrams

**Difficulty:** Medium
**Tags:** hashing, string

## Problem

Given a list of strings, group the anagrams together. You can return the groups in any order.

## Example

```
Input: strs = ["eat","tea","tan","ate","nat","bat"]
Output: [["bat"],["nat","tan"],["ate","eat","tea"]]   # any grouping/order is fine
```

## Constraints

- `1 <= len(strs) <= 10^4`
- `0 <= len(strs[i]) <= 100`
- `strs[i]` consists of lowercase English letters.

## Hints

1. Two strings are anagrams if and only if some canonical form of them is identical. What's a cheap canonical form?
2. Sorting the characters of a string gives a canonical key: `"eat"` and `"tea"` both sort to `"aet"`.
3. Use a dictionary mapping `sorted_key -> list of original strings`, then return `dict.values()`. For long strings, a character-count tuple is a faster key than sorting.
