# Binary Tree Level Order Traversal

**Difficulty:** Medium
**Tags:** tree, bfs, queue

## Problem

Given the root of a binary tree, return the level-order traversal of its node values (i.e. from left to right, level by level), as a list of lists — one inner list per level.

Assume a simple node class:
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

## Example

```
Input tree:
    3
   / \
  9  20
    /  \
   15   7

Output: [[3], [9, 20], [15, 7]]
```

## Constraints

- Number of nodes is in range `[0, 2000]`.
- `-1000 <= Node.val <= 1000`

## Hints

1. "Level by level" strongly suggests breadth-first traversal rather than depth-first.
2. Use a queue (e.g. `collections.deque`), starting with the root. The key trick is processing one full level at a time.
3. At each iteration, record the current queue length as the number of nodes in this level, pop exactly that many, collect their values, and enqueue their children — that batch becomes one entry in your result list.
