# Longest Common Subsequence

**Difficulty:** Medium
**Tags:** dynamic-programming, string

## Problem

Given two strings `text1` and `text2`, return the length of their longest common subsequence (a subsequence need not be contiguous, but must preserve relative order). If there is no common subsequence, return 0.

## Example

```
Input: text1 = "abcde", text2 = "ace"
Output: 3        # "ace"

Input: text1 = "abc", text2 = "abc"
Output: 3

Input: text1 = "abc", text2 = "def"
Output: 0
```

## Constraints

- `1 <= len(text1), len(text2) <= 1000`
- Strings consist of lowercase English letters.

## Hints

1. Comparing character by character with plain recursion re-solves the same subproblems repeatedly — a strong hint for a 2D DP table.
2. Define `dp[i][j]` = length of LCS of `text1[:i]` and `text2[:j]`. What happens when `text1[i-1] == text2[j-1]` versus when they differ?
3. If the characters match: `dp[i][j] = dp[i-1][j-1] + 1`. If not: `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`. Base case: `dp[0][*] = dp[*][0] = 0`.
