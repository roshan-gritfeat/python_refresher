# Number of Islands

**Difficulty:** Medium
**Tags:** graph, dfs/bfs, matrix

## Problem

Given an `m x n` 2D binary grid `grid` which represents a map of `'1'`s (land) and `'0'`s (water), return the number of islands. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically (not diagonally).

## Example

```
Input: grid = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
Output: 3
```

## Constraints

- `1 <= m, n <= 300`
- `grid[i][j]` is `'0'` or `'1'`.

## Hints

1. Whenever you find an unvisited land cell, everything connected to it (up/down/left/right) is part of the same island — think of this as a graph connectivity problem.
2. Scan every cell; when you find a `'1'` you haven't visited, that's a new island — increment your count and then "sink" the whole island so you don't count it again.
3. Use DFS or BFS (recursive DFS is simplest) to mark every connected land cell as visited (e.g. by mutating the grid to `'0'`, or using a separate visited set).
