# Number of Connected Components (Union-Find)

**Difficulty:** Hard
**Tags:** graph, union-find

## Problem

You have `n` nodes labeled `0` to `n-1` and a list of undirected edges. Using a **Union-Find (Disjoint Set Union)** data structure that you implement yourself (no using DFS/BFS as the primary mechanism — the point is to practice DSU), return the number of connected components in the graph. As a follow-up, also support an `is_connected(a, b)` query after processing all edges.

## Example

```
Input: n = 5, edges = [[0,1],[1,2],[3,4]]
Output: 2       # components: {0,1,2} and {3,4}
```

## Constraints

- `1 <= n <= 2 * 10^5`
- `0 <= len(edges) <= 2 * 10^5`

## Hints

1. Union-Find maintains a "parent" array; two nodes are in the same component if they share the same root parent (found by following parent pointers up).
2. Implement `find(x)` (follow parents to the root, ideally with **path compression** — point nodes directly at the root as you go) and `union(a, b)` (attach one root under the other).
3. Use **union by rank/size** (always attach the smaller tree under the bigger one's root) combined with path compression to get near-O(1) amortized operations. Track the component count, decrementing it each time a `union` actually merges two previously-separate roots.
