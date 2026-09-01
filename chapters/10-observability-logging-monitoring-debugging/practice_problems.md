# Chapter 10 — Practice Problems: Observability, Logging & Debugging

Standalone exercises, ordered easy to hard.

1. **Configure stdlib `logging` to output JSON.** Write a script that logs three different events at three different levels, formatted as one JSON object per line.
   *Focus: custom `logging.Formatter`, understanding `%(levelname)s`-style fields.*

2. **Switch the same script to `structlog`.** Reproduce the same three log events using `structlog.get_logger()` and `log.bind(...)`.
   *Hint: compare how context accumulation differs from manually formatting strings.*

3. **Add a correlation ID with `contextvars`.** Write a small async function that spawns three concurrent "requests" (via `asyncio.gather`), each setting its own correlation ID in a `ContextVar`, and prove that logs from each never leak into another's context.
   *Focus: why a plain global variable would fail this test under concurrency.*

4. **Build a minimal FastAPI middleware** that reads an `X-Request-ID` header if present, generates a UUID otherwise, and echoes it back in the response headers.
   *Hint: `@app.middleware("http")`, `call_next`.*

5. **Implement a `/health` endpoint** for a toy FastAPI app that checks a (mock) database connection and a (mock) Redis connection, returning `503` if either is unreachable and `200` otherwise.
   *Focus: difference between an "always 200" health check and one that verifies real dependencies.*

6. **Expose a `/metrics` endpoint** using `prometheus_client` that tracks a counter of total requests and a histogram of request duration for a toy Flask or FastAPI app.
   *Hint: `Counter`, `Histogram`, and `generate_latest()`.*

7. **Profile a deliberately slow function with `cProfile`.** Write a function that does an unnecessary O(n²) string concatenation in a loop over 50,000 items, profile it, and identify the hot spot from the `pstats` output.
   *Focus: reading cumulative time vs total time in profiler output.*

8. **Fix the function from #7** using a more efficient approach (e.g., `str.join`) and re-profile to confirm the improvement, reporting the speedup factor.
   *Hint: measure with `time.perf_counter()` before/after in addition to the profiler.*

9. **Attach `py-spy` to a running process.** Start a long-running toy script in one terminal, use `py-spy top --pid <pid>` (or `py-spy dump`) from another to inspect its live call stack without modifying the script.
   *Focus: understand this requires no code changes, unlike `cProfile`.*

10. **Find a memory leak with `tracemalloc`.** Write a script with a module-level list that a function accidentally appends to on every call (instead of a local list), take two `tracemalloc` snapshots around repeated calls, and diff them to locate the leaking allocation site.
    *Hint: `tracemalloc.take_snapshot()` and `snapshot.compare_to(other, "lineno")`.*

11. **Simulate log-based debugging.** Given a synthetic log file (write one yourself) containing JSON lines from multiple interleaved correlation IDs, write a script that filters and reconstructs the ordered sequence of events for one specific correlation ID.
    *Focus: this is exactly the manual process you'd do against a real log aggregator.*

12. **Design (on paper/markdown, no code required) a minimal distributed trace** for a hypothetical request that touches an API service, a database call, and an external HTTP call to a third-party service — sketch the spans, their parent/child relationship, and durations.
    *Focus: understanding spans and parent-child relationships before reaching for OpenTelemetry tooling.*
