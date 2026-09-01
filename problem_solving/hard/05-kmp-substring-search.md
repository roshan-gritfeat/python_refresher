# KMP Substring Search

**Difficulty:** Hard
**Tags:** string-matching, kmp

## Problem

Given a text string `text` and a pattern string `pattern`, find the starting indices of all occurrences of `pattern` in `text`, using the **Knuth-Morris-Pratt (KMP)** algorithm (i.e. achieve O(len(text) + len(pattern)) time — no relying on `str.find` in a loop or naive O(n*m) matching).

## Example

```
Input: text = "ababcabcabababd", pattern = "abab"
Output: [0, 7, 10]
```

## Constraints

- `1 <= len(pattern) <= len(text) <= 10^6`
- Strings consist of printable ASCII characters.

## Hints

1. The naive approach re-checks characters it already matched when a mismatch occurs partway through — KMP's insight is to never re-examine text characters by precomputing pattern self-similarity.
2. First, build the "longest proper prefix that is also a suffix" (LPS / failure function) array for the pattern. `lps[i]` tells you, for `pattern[0..i]`, the length of the longest prefix of `pattern` that is also a suffix ending at `i`.
3. While scanning `text` with a pattern pointer `j`: on a match, advance both pointers; on a mismatch with `j > 0`, don't reset `j` to 0 — instead set `j = lps[j-1]` and retry, since that much of the pattern is guaranteed to already match. When `j` reaches `len(pattern)`, record a match and continue with `j = lps[j-1]`.
