# Implement Queue Using Two Stacks

**Difficulty:** Easy
**Tags:** stack, queue, design

## Problem

Implement a first-in-first-out (FIFO) queue using only two stacks. The implemented queue should support `enqueue(x)`, `dequeue()` (removes and returns the front element), `peek()` (returns the front element without removing it), and `is_empty()`.

## Example

```
q = MyQueue()
q.enqueue(1)
q.enqueue(2)
q.peek()      # returns 1
q.dequeue()   # returns 1
q.is_empty()  # returns False
q.dequeue()   # returns 2
q.is_empty()  # returns True
```

## Constraints

- Use only stack operations (`append`/`pop` from the end of a Python list) — no `list.pop(0)` or `collections.deque` shortcuts that give you a queue directly.
- At most `10^4` operations will be called.

## Hints

1. One stack alone reverses order incorrectly for FIFO — think about what a *second* stack, used to reverse the reversal, buys you.
2. Use an "in" stack for enqueuing and an "out" stack for dequeuing/peeking.
3. When the "out" stack is empty and you need to dequeue or peek, pop everything from "in" and push it onto "out" — this reverses the order so the oldest element ends up on top of "out".
