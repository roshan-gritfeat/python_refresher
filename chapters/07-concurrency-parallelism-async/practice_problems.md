# Chapter 7 — Practice Problems

Standalone exercises, independent of TaskFlow. Ordered easy to hard.

1. **Diagnose the bottleneck** — Given two functions, one that sleeps for 2 seconds and one that computes `sum(i*i for i in range(50_000_000))`, state which is I/O-bound and which is CPU-bound, and which concurrency tool would help each.
   *Focus: no code needed — this is a diagnosis exercise. Explain your reasoning.*

2. **Threads for I/O** — Write a script that downloads (or simulates via `time.sleep`) 5 "URLs" sequentially, then rewrite it using `threading.Thread` to run concurrently, and print the wall-clock time for both.
   *Hint: use `time.perf_counter()` before/after; expect the threaded version close to the slowest single task, not the sum.*

3. **`ThreadPoolExecutor` basics** — Redo problem 2 using `concurrent.futures.ThreadPoolExecutor.map` instead of raw `threading.Thread`.
   *Focus: notice how much less boilerplate this requires versus manual thread management.*

4. **First asyncio program** — Write an `async def fetch(n)` that `await asyncio.sleep(n)` then returns `n * 2`, and use `asyncio.gather` to run it concurrently for `n in [1, 2, 3]`, confirming total runtime is ~3s not 6s.
   *Hint: remember to run the top-level coroutine with `asyncio.run(...)`.*

5. **The blocking-call trap** — Take the script from problem 4 and replace one `await asyncio.sleep(n)` with a plain `time.sleep(n)` inside the same coroutine. Observe and explain why total runtime changes.
   *Focus: this demonstrates how a single blocking call stalls the entire event loop.*

6. **CPU-bound with multiprocessing** — Write a function that computes whether each number in a list of 20 large numbers is prime (naively, no early tricks), run it single-threaded and timed, then rerun using `multiprocessing.Pool` (or `ProcessPoolExecutor`) and compare wall-clock time.
   *Hint: use numbers large enough that the naive check takes real time (a few hundred milliseconds each) so the difference is visible.*

7. **Threads don't help CPU-bound work** — Take the prime-checking function from problem 6 and instead parallelize it with `ThreadPoolExecutor`. Compare its timing to both the single-threaded and multiprocessing versions.
   *Focus: confirm empirically that threads don't speed up CPU-bound pure-Python work, and explain why using the GIL.*

8. **Reproduce a race condition** — Write a shared counter incremented 100,000 times by each of 4 threads without a lock, run it several times, and show the final value is inconsistent and less than the expected 400,000.
   *Hint: run it in a loop 5 times and print the result each time to make the non-determinism visible.*

9. **Fix it with a lock** — Fix problem 8 using `threading.Lock`, and confirm the result is now always exactly 400,000 across repeated runs.
   *Focus: identify the minimal critical section — don't hold the lock longer than necessary.*

10. **Producer-consumer with a queue** — Build a producer thread that pushes 50 numbers onto a `queue.Queue` and 3 consumer threads that pop and "process" (e.g., square) them, using a sentinel value to signal shutdown cleanly.
    *Hint: think carefully about how many sentinels you need if there are multiple consumers.*

11. **Async fan-out with error handling** — Write an async function that fetches from 5 "URLs" where one deliberately raises an exception, and use `asyncio.gather(..., return_exceptions=True)` to collect both successes and the exception without aborting the others.
    *Focus: inspect the returned list and distinguish real results from `Exception` instances.*

12. **Deadlock, then fix it** — Write two threads that each try to acquire two locks in opposite order (thread A: lock1 then lock2; thread B: lock2 then lock1) and demonstrate a deadlock (or a near-hang). Then fix it by enforcing a consistent lock acquisition order across both threads.
    *Hint: add a timeout to `Lock.acquire(timeout=...)` first to observe the deadlock without hanging your terminal forever, then apply the real fix.*
