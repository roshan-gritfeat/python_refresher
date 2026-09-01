# Edit Distance

**Difficulty:** Hard
**Tags:** dynamic-programming, string

## Problem

Given two strings `word1` and `word2`, return the minimum number of operations required to convert `word1` into `word2`. You have three operations permitted on `word1`: insert a character, delete a character, or replace a character.

## Example

```
Input: word1 = "horse", word2 = "ros"
Output: 3     # horse -> rorse (replace h->r) -> rose (delete r) -> ros (delete e)

Input: word1 = "intention", word2 = "execution"
Output: 5
```

## Constraints

- `0 <= len(word1), len(word2) <= 500`
- Lowercase English letters.

## Hints

1. At any pair of positions `(i, j)` in the two words, you have three possible "last operation" choices (insert, delete, replace) plus the case where the characters already match — this recursive structure suggests a 2D DP table, similar in spirit to Longest Common Subsequence.
2. Define `dp[i][j]` = edit distance between `word1[:i]` and `word2[:j]`. Base cases: `dp[i][0] = i` (delete all), `dp[0][j] = j` (insert all).
3. If `word1[i-1] == word2[j-1]`: `dp[i][j] = dp[i-1][j-1]` (no operation needed). Otherwise: `dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])` (delete, insert, replace respectively).
