# Chapter 7 — Concurrency, Parallelism & Async

Goal: understand why your code is slow and which tool actually fixes it.

## The GIL: what it actually restricts and doesn't

The **Global Interpreter Lock (GIL)** is a mutex in CPython that ensures only one thread executes Python bytecode at a time, even on a multi-core machine. This is a common source of confusion: people hear "Python doesn't have real threads" and overcorrect into thinking threads are useless in Python. That's wrong.

What the GIL restricts: **CPU-bound** work (tight loops doing arithmetic, string processing, image manipulation in pure Python) does not get faster by adding threads, because only one thread runs Python bytecode at any instant — you get concurrency (interleaving) but not parallelism (simultaneous execution) for that kind of work.

What the GIL does *not* restrict: **I/O-bound** work. When a thread is waiting on a network call, disk read, or `time.sleep`, it releases the GIL, letting other threads run. So threads are genuinely useful for I/O concurrency in Python — many libraries implemented in C (e.g., `numpy`, `hashlib` for large inputs) also release the GIL during their heavy-lifting, allowing real parallelism for those operations even in threads.

**Common pitfall:** assuming "the GIL means Python has no concurrency benefit from threads" — this is only true for CPU-bound pure-Python code. Also worth knowing: Python 3.13 introduced an experimental free-threaded (no-GIL) build; it's not yet the default and ecosystem compatibility is still catching up, but it signals where CPython is heading.

```python
import threading, time

def io_task():
    time.sleep(1)  # releases the GIL while waiting

start = time.perf_counter()
threads = [threading.Thread(target=io_task) for _ in range(5)]
[t.start() for t in threads]
[t.join() for t in threads]
print(time.perf_counter() - start)  # ~1s, not 5s — I/O concurrency works
```

## `threading` vs `multiprocessing` vs `asyncio` — when each is the right tool

- **`threading`**: lightweight, shares memory with the parent process (be careful — this is exactly what causes race conditions), good for I/O-bound work with a *moderate* number of concurrent operations, and for using existing synchronous, blocking libraries concurrently without rewriting them as async.
- **`multiprocessing`**: spawns separate OS processes, each with its own Python interpreter and GIL, so it achieves real parallelism for **CPU-bound** work. Costs more overhead (process startup, inter-process communication via pickling) and memory (no shared state by default — you use `multiprocessing.Queue`, `Manager`, or shared memory explicitly).
- **`asyncio`**: single-threaded cooperative concurrency via an event loop. Extremely efficient for **I/O-bound** work with *many* concurrent operations (thousands of open connections), because there's no per-task OS thread overhead. Requires the entire call chain to be async-aware (`async def`, `await`) — mixing in a blocking call stalls the whole event loop.

**Decision rule:** ask "is this task limited by waiting on something external (I/O-bound) or limited by the CPU actually crunching (CPU-bound)?" I/O-bound with few tasks → threading is simplest. I/O-bound with many tasks or needing fine control over cancellation/timeouts → asyncio. CPU-bound → multiprocessing (or move the hot loop into a C-extension library like `numpy` that releases the GIL).

**Common pitfall:** reaching for `asyncio` for CPU-bound work (it won't help — a synchronous CPU-heavy function in an async app blocks the entire event loop) or reaching for `multiprocessing` for I/O-bound work (correct but wasteful — you pay process overhead for something threads or asyncio would handle far more cheaply).

## `asyncio` fundamentals: event loop, coroutines, `async`/`await`, `gather`, tasks

The **event loop** is a single-threaded scheduler that runs coroutines, switching between them whenever one hits an `await` on something not yet ready (a socket read, a timer, etc.). A **coroutine** (defined with `async def`) is a function that can suspend and resume; calling it doesn't run it immediately — it produces a coroutine object that must be awaited or scheduled.

```python
import asyncio

async def fetch(url: str) -> str:
    await asyncio.sleep(0.5)  # simulate network I/O
    return f"data from {url}"

async def main():
    results = await asyncio.gather(fetch("a"), fetch("b"), fetch("c"))
    print(results)

asyncio.run(main())  # ~0.5s total, not 1.5s
```

`asyncio.gather` runs multiple awaitables concurrently and collects their results (or raises the first exception, by default cancelling the rest — pass `return_exceptions=True` to instead collect exceptions alongside results). A **Task** (`asyncio.create_task(coro)`) schedules a coroutine to run concurrently on the event loop *immediately*, rather than waiting for you to `await` it — useful when you want to fire off work and check on it later.

**Common pitfalls:**
- Calling a blocking function (e.g., `requests.get`, `time.sleep`, a synchronous DB driver) inside an `async def` — this blocks the *entire* event loop, stalling every other coroutine, not just the caller.
- Forgetting to `await` a coroutine — Python will warn "coroutine was never awaited," and the code silently does nothing.
- Not handling task cancellation / exceptions — an exception in a `create_task`'d task that's never awaited or checked can be swallowed silently until garbage collection.

## Async web servers and async DB drivers

An **ASGI** server (Uvicorn, Hypercorn) is what lets FastAPI (Chapter 6) handle many concurrent requests on one thread when your endpoint handlers are `async def` and use non-blocking I/O throughout the stack. The moment one handler calls a *synchronous* DB driver or does blocking file I/O directly in an `async def`, that call blocks the whole event loop for everyone.

This is why **async DB drivers** matter: SQLAlchemy's default synchronous engine (`sqlite3`, `psycopg2`) blocks. To get real async benefits with SQLAlchemy, you switch to an async engine (`aiosqlite` for SQLite, `asyncpg`/`psycopg` async mode for Postgres) and use `AsyncSession` with `await session.execute(...)`.

**Common pitfall:** adding `async def` to route handlers while still using the synchronous SQLAlchemy engine underneath — this looks async but gains nothing, because FastAPI will actually run a genuinely-sync dependency in a threadpool (fine) but a sync call *inside* an `async def` handler body runs on the event loop thread and blocks it (not fine). Be deliberate about which one you're doing.

## `concurrent.futures` for simpler thread/process pools

`concurrent.futures.ThreadPoolExecutor` and `ProcessPoolExecutor` provide a simpler, higher-level API than raw `threading`/`multiprocessing` for the common "run N independent tasks and collect results" pattern.

```python
from concurrent.futures import ThreadPoolExecutor

def fetch_sync(url: str) -> str:
    ...  # blocking network call

with ThreadPoolExecutor(max_workers=5) as pool:
    results = list(pool.map(fetch_sync, urls))
```

Swap `ThreadPoolExecutor` for `ProcessPoolExecutor` to run CPU-bound functions in parallel processes instead — same interface, very different execution model underneath. This uniformity is exactly why it's worth learning: you can benchmark both with a one-line swap and empirically decide which fits.

**Common pitfall:** using `ProcessPoolExecutor` with non-picklable arguments/return values (lambdas, open file handles, DB connections) — everything sent to and from a worker process must be pickled, and this is a frequent, confusing failure mode.

## Race conditions, locks, deadlocks — and how to avoid them

A **race condition** occurs when multiple threads (or processes) read and modify shared state without coordination, and the outcome depends on timing. The classic example is a non-atomic increment:

```python
counter = 0

def increment():
    global counter
    for _ in range(100_000):
        counter += 1  # read-modify-write: NOT atomic

# Run this in two threads and counter often ends up < 200_000
```

`counter += 1` is actually three steps (read, add, write); if two threads interleave between the read and write, one increment is lost. The fix is a **lock**:

```python
import threading
lock = threading.Lock()

def increment_safe():
    global counter
    for _ in range(100_000):
        with lock:
            counter += 1
```

A **deadlock** happens when two or more threads each hold a lock the other needs, and neither can proceed — classically, thread A holds lock 1 and waits for lock 2, while thread B holds lock 2 and waits for lock 1. Avoid deadlocks by always acquiring multiple locks in a consistent global order, keeping critical sections small, and preferring higher-level primitives (`Queue`, `asyncio.Lock`) over manual fine-grained locking where possible.

**Common pitfalls:**
- Assuming a single line of Python is atomic just because it's one line — bytecode-level interleaving can still happen mid-statement.
- Holding a lock across a blocking I/O call, needlessly serializing work that didn't need to be serialized.
- Using a `threading.Lock` where you meant `asyncio.Lock` (or vice versa) — they are not interchangeable across the sync/async boundary.

## Queues for producer-consumer patterns

A **queue** decouples producers (things generating work) from consumers (things processing it), letting each side run at its own pace. `queue.Queue` (thread-safe) is used with `threading`; `multiprocessing.Queue` for processes; `asyncio.Queue` for coroutines.

```python
import queue, threading

q = queue.Queue(maxsize=10)

def producer():
    for item in range(20):
        q.put(item)
    q.put(None)  # sentinel to signal completion

def consumer():
    while (item := q.get()) is not None:
        process(item)
```

**Why it matters:** without a queue, producers and consumers must run in lockstep or you need ad-hoc coordination (shared lists with locks, busy-waiting) that's both slower and more error-prone. Queues also naturally provide **backpressure** — a bounded queue's `put()` blocks once full, preventing an over-eager producer from exhausting memory.

**Common pitfalls:**
- Forgetting a way to signal "no more work" (a sentinel value, or `queue.join()`/`task_done()` bookkeeping) — consumers can hang forever waiting on an empty queue.
- Using an unbounded queue for unbounded producers, silently growing memory usage instead of applying backpressure.
