# Word Search

**Difficulty:** Medium
**Tags:** backtracking, matrix, dfs

## Problem

Given an `m x n` grid of characters `board` and a string `word`, return `True` if `word` exists in the grid. The word can be constructed from letters of sequentially adjacent cells (horizontally or vertically neighboring), and the same cell may not be used more than once within one word.

## Example

```
Input: board = [["A","B","C","E"],
                ["S","F","C","S"],
                ["A","D","E","E"]], word = "ABCCED"
Output: True

Input: board = [...same...], word = "SEE"
Output: True

Input: board = [...same...], word = "ABCB"
Output: False
```

## Constraints

- `1 <= m, n <= 6`
- `1 <= len(word) <= 15`

## Hints

1. This is a search problem: from each starting cell, try to trace the word letter by letter through neighboring cells — classic backtracking territory.
2. For each cell matching `word[0]`, do a DFS trying to match subsequent characters in the four directions, marking cells as visited during the current path.
3. Crucially, un-mark (backtrack) a cell after exploring it so it can be reused in a different path attempt. A quick early-exit: if a letter in `word` doesn't appear in `board` at all, or letter frequency in `board` can't cover `word`, you can return `False` immediately.
