# Course Schedule

**Difficulty:** Medium
**Tags:** graph, topological-sort, cycle-detection

## Problem

There are `numCourses` courses labeled `0` to `numCourses - 1`. You are given a list of prerequisite pairs `[a, b]` meaning you must take course `b` before course `a`. Return `True` if it's possible to finish all courses, `False` otherwise.

## Example

```
Input: numCourses = 2, prerequisites = [[1,0]]
Output: True    # take 0 then 1

Input: numCourses = 2, prerequisites = [[1,0],[0,1]]
Output: False   # cycle: 0 needs 1, 1 needs 0
```

## Constraints

- `1 <= numCourses <= 2000`
- `0 <= len(prerequisites) <= 5000`

## Hints

1. This is really asking: "does the prerequisite graph contain a cycle?" If it does, you can never finish; if not, a valid order (topological sort) exists.
2. Build an adjacency list and compute each node's in-degree (number of prerequisites). Nodes with in-degree 0 can be taken immediately.
3. Use Kahn's algorithm: repeatedly remove in-degree-0 nodes, decrementing the in-degree of their neighbors. If you can remove all `numCourses` nodes this way, there's no cycle. (Alternatively, do DFS with a "currently in recursion stack" marker to detect back-edges.)
