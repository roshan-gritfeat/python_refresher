# Dijkstra's Shortest Path

**Difficulty:** Hard
**Tags:** graph, dijkstra, heap

## Problem

Given a weighted, directed graph with `n` nodes (labeled `0` to `n-1`), a list of edges `[u, v, weight]` (all weights non-negative), and a source node `src`, return an array `dist` where `dist[i]` is the length of the shortest path from `src` to node `i` (or `float('inf')` if unreachable).

## Example

```
Input: n = 5, edges = [[0,1,4],[0,2,1],[2,1,2],[1,3,1],[2,3,5],[3,4,3]], src = 0
Output: dist = [0, 3, 1, 4, 7]
```

## Constraints

- `1 <= n <= 10^4`
- `0 <= len(edges) <= 5 * 10^4`
- `0 <= weight <= 10^4`
- No negative weights.

## Hints

1. BFS finds shortest paths for unweighted graphs; here edges have different costs, so you need to always expand the currently-closest unvisited node next.
2. A min-heap (priority queue) keyed by current known distance is the standard tool — this is essentially a greedy BFS ordered by distance.
3. Push `(0, src)` onto the heap. Repeatedly pop the smallest-distance node; if it's already finalized (visited), skip it; otherwise finalize it and relax (try to improve) all its neighbors' distances, pushing any improvements onto the heap. Watch out for stale heap entries — you may push a node multiple times with different distances before it's finalized.
