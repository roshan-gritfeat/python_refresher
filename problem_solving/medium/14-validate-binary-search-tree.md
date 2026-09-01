# Validate Binary Search Tree

**Difficulty:** Medium
**Tags:** tree, recursion

## Problem

Given the root of a binary tree, determine if it is a valid binary search tree (BST). A valid BST is defined as: the left subtree of a node contains only nodes with values strictly less than the node's value, the right subtree only nodes with values strictly greater, and both subtrees must also be valid BSTs.

## Example

```
Input:
    2
   / \
  1   3
Output: True

Input:
    5
   / \
  1   4
     / \
    3   6
Output: False   # 4 is in the right subtree of 5, so everything under it must be > 5,
                # but 4's own left child is 3, and more importantly 4 itself is not > 5
```

## Constraints

- Number of nodes is in the range `[1, 10^4]`.
- `-2^31 <= Node.val <= 2^31 - 1`

## Hints

1. It's not enough to just check `node.left.val < node.val < node.right.val` locally — a node deep in the left subtree must be less than ALL of its ancestors up the chain, not just its immediate parent.
2. Pass down a valid `(low, high)` range as you recurse; a node must fall strictly within that range.
3. When recursing left, tighten the upper bound to the current node's value; when recursing right, tighten the lower bound. Start with `(-infinity, +infinity)`. An in-order traversal that checks strictly increasing values is an alternative approach.
