# N-Queens

**Difficulty:** Hard
**Tags:** backtracking

## Problem

Place `n` queens on an `n x n` chessboard such that no two queens attack each other (no shared row, column, or diagonal). Return all distinct solutions, where each solution is a list of strings representing the board (`'Q'` for a queen, `'.'` for empty).

## Example

```
Input: n = 4
Output: [
  [".Q..","...Q","Q...","..Q."],
  ["..Q.","Q...","...Q",".Q.."]
]     # 2 distinct solutions for n=4
```

## Constraints

- `1 <= n <= 9`

## Hints

1. Since no two queens can share a row, you can place exactly one queen per row and just decide which column it goes in for each row — this cuts the search space dramatically.
2. Use backtracking: try each column in the current row; if it's safe (no conflict with previously placed queens), place it and recurse to the next row; otherwise skip.
3. To check "safe" in O(1) instead of re-scanning placed queens each time, track three sets: used columns, used "diagonals" (`row - col` is constant along one diagonal direction), and used "anti-diagonals" (`row + col` is constant along the other direction).
