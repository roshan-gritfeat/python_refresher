# Thread-Safe Bounded Blocking Queue

**Difficulty:** Hard
**Tags:** concurrency, design

## Problem

Implement a bounded blocking queue (a classic producer-consumer primitive) supporting:

- `put(item)` — adds an item to the queue; if the queue is full, the calling thread should **block** until space becomes available.
- `get()` — removes and returns an item; if the queue is empty, the calling thread should **block** until an item becomes available.

The queue must behave correctly when multiple producer and consumer threads call `put`/`get` concurrently — no lost updates, no deadlocks, no busy-waiting (don't just spin-loop checking a condition).

## Example

```python
q = BoundedBlockingQueue(capacity=2)

# Producer thread:
q.put(1)
q.put(2)
q.put(3)   # blocks here until a consumer calls get()

# Consumer thread (running concurrently):
q.get()    # returns 1, unblocks the producer's third put()
```

## Constraints

- Must work correctly under Python's threading model (be explicit about what `threading` primitives you use — this problem is about correct synchronization, not about circumventing the GIL).
- No busy-waiting / polling loops with `sleep` — use proper condition signaling.

## Hints

1. This is the textbook producer-consumer problem. You need mutual exclusion around the internal buffer, plus a way for threads to sleep efficiently until a condition (not full / not empty) becomes true.
2. Python's `threading.Condition` is built exactly for this: it pairs a lock with `wait()` (release the lock and sleep until notified) and `notify()`/`notify_all()` (wake waiters).
3. In `put`, acquire the condition, `while len(buffer) == capacity: cond.wait()`, then append and `cond.notify_all()` (to wake any waiting consumers). Mirror this in `get`, waiting `while len(buffer) == 0`, then popping and notifying waiting producers. Use a single shared `Condition` (or two, carefully) to avoid missed-wakeup races — always check the condition in a `while` loop, never a bare `if`, to protect against spurious wakeups.
