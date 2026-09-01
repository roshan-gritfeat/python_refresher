# Count Vowels and Consonants

**Difficulty:** Easy
**Tags:** string, counting

## Problem

Given a string composed of English letters (and possibly spaces/punctuation), count the number of vowels and the number of consonants. Treat `a, e, i, o, u` (case-insensitive) as vowels; ignore non-letter characters entirely.

## Example

```
Input: "Hello, World!"
Output: {"vowels": 3, "consonants": 7}
```

## Constraints

- `0 <= len(s) <= 10^5`
- `s` may contain letters, digits, punctuation, and whitespace.

## Hints

1. Normalize the string to lowercase first so you only need to check one case.
2. Use `str.isalpha()` to skip non-letter characters.
3. Keep a set of vowel characters for O(1) membership checks: `{'a','e','i','o','u'}`.
