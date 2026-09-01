# Chapter 7 Project — TaskFlow: Concurrency, Parallelism & Async

## Where TaskFlow stands entering this chapter

By the end of Chapter 6, TaskFlow is a FastAPI application backed by the Chapter 5 SQLAlchemy/SQLite persistence layer and Redis cache, with JWT-protected CRUD endpoints for tasks, request/response validation via Pydantic, consistent error responses, and reviewed OpenAPI docs — all still covered by the pytest suite, ruff/mypy, and pre-commit from earlier chapters.

## What to add this chapter

Identify real I/O-bound and CPU-bound pain points in TaskFlow, fix the I/O-bound one with `asyncio`, fix the CPU-bound one with `multiprocessing`, and deliberately create and fix a race condition — all with before/after benchmarks proving the change mattered.

### 1. Convert an I/O-heavy feature to asyncio

- Add a feature that is genuinely I/O-bound and currently synchronous: e.g., a "task enrichment" step that calls 3+ external services concurrently per task (simulate with `httpx` calls to a public API, or a local mock server / `asyncio.sleep`-based stub if you want to avoid external dependencies) — for example, fetching a weather forecast, a motivational quote, and a due-date holiday check to attach to a task.
- Write the synchronous version first (sequential `requests` calls) and benchmark it over, say, 10 tasks needing enrichment.
- Rewrite it using `async def` + `httpx.AsyncClient` + `asyncio.gather`, and benchmark again.
- If your FastAPI endpoints were still using the synchronous SQLAlchemy engine, this is also the moment to evaluate switching the relevant session/engine to an async driver (`aiosqlite`) so the async endpoint doesn't block on synchronous DB calls — document your decision either way.
- Record both benchmark numbers (e.g., in a `BENCHMARKS.md` or in the module docstring) and explain the speedup in terms of I/O-bound vs CPU-bound.

### 2. Add a CPU-bound multiprocessing task

- Add a genuinely CPU-bound feature: a "bulk task analytics" job that processes a large batch of tasks (e.g., 50,000+ synthetic tasks) computing something non-trivial per task — word-frequency analysis on task descriptions, or a scoring/prioritization algorithm with real computation.
- Implement it single-threaded first and benchmark.
- Reimplement using `multiprocessing.Pool` or `concurrent.futures.ProcessPoolExecutor`, partitioning the batch across workers, and benchmark again.
- Expose it via a TaskFlow CLI command or an admin API endpoint (e.g., `POST /admin/analytics/recompute`), being explicit that this endpoint should not block the async event loop — run it in a process pool via `loop.run_in_executor` if triggered from an async endpoint.

### 3. Create and fix a race condition

- Identify (or introduce) genuinely shared mutable state in TaskFlow that multiple threads/tasks touch concurrently — e.g., an in-memory request counter, a "tasks completed today" tally cached outside the DB, or a naive in-process rate limiter.
- Write a test or script that runs many concurrent increments/updates against it and demonstrates the race condition (lost updates, inconsistent totals) — capture this as evidence (e.g., a failing assertion or printed inconsistent totals across runs).
- Fix it with an appropriate lock (`threading.Lock` for thread-based access, or restructure to rely on Redis's atomic operations like `INCR` if it fits the domain better — Redis is already in TaskFlow from Chapter 5).
- Add a regression test that fails without the fix and passes with it (may need to run with real threads and enough iterations to make the race reliably observable).

### 4. Document the decisions

- Add a short section (README or docstring) to TaskFlow explaining, for each of the three changes, why that specific tool (asyncio / multiprocessing / locks) was the right one — tie it back to the I/O-bound vs CPU-bound framework from this chapter.

## Definition of done

- [ ] An I/O-heavy TaskFlow feature has both a synchronous and an asyncio-based implementation, with measured benchmark numbers showing the speedup.
- [ ] A CPU-bound bulk-processing feature has both a single-threaded and a multiprocessing-based implementation, with measured benchmark numbers showing the speedup.
- [ ] The CPU-bound feature is exposed without blocking the FastAPI event loop when triggered from an async endpoint.
- [ ] A race condition has been deliberately reproduced (with evidence of inconsistent results) and then fixed with a lock or atomic Redis operation.
- [ ] A regression test exists that fails on the unfixed race condition and passes on the fixed version.
- [ ] Benchmark results and the reasoning behind each tool choice (asyncio vs multiprocessing vs locking) are documented in the repo.
- [ ] Existing pytest suite, ruff, and mypy checks still pass; new code is covered by tests.
