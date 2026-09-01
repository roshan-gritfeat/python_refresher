# Chapter 10 — Observability: Logging, Monitoring & Debugging in Production

**Goal:** know what your code is doing when you're not watching it. A deployed service (as of Chapter 9, TaskFlow is now live on the internet) fails silently unless you deliberately build the ability to see inside it.

---

## Structured logging

### `structlog` or stdlib `logging` with JSON formatting

Traditional logging produces free-text lines like `Task created for user 42`. That's fine for a human tailing a terminal, but useless for a machine — a log aggregation system (Datadog, ELK, CloudWatch Logs Insights) can't reliably query "give me all failed task creations for user 42" out of unstructured prose. **Structured logging** emits each log entry as a set of key-value fields — typically serialized as JSON — so every field is independently searchable and aggregable:

```json
{"timestamp": "2026-08-31T10:00:00Z", "level": "info", "event": "task_created", "user_id": 42, "task_id": 981, "correlation_id": "a1b2c3"}
```

Python's stdlib `logging` module can be configured with a custom `Formatter` that outputs JSON, but `structlog` is purpose-built for this: it treats log calls as building up a dictionary of context (`log.bind(user_id=42)`) rather than formatting a string immediately, and lets you attach "processors" (chains of functions) that add timestamps, render JSON, filter by level, etc. The core mental model shift: **you don't write a log message, you record an event with attributes.**

Common pitfalls: logging at the wrong level (using `INFO` for what's really a warning, or `ERROR` for expected/handled conditions — this trains engineers to ignore alerts), and interpolating sensitive data (passwords, tokens, full request bodies) directly into log fields without redaction.

### Log levels

The standard hierarchy — `DEBUG < INFO < WARNING < ERROR < CRITICAL` — exists so you can tune verbosity per environment without changing code: run `DEBUG` locally to see everything, run `INFO` or `WARNING` in production so log volume (and cost) stays manageable, and rely on `ERROR`/`CRITICAL` to page a human. A frequent pitfall is "log level inflation" — everything becomes `ERROR` because a developer wants to make sure it's seen, which defeats the entire purpose of levels and causes alert fatigue.

### Correlation IDs

A single user request to TaskFlow might touch the API layer, the service layer, the repository layer, and a Redis cache lookup — potentially across multiple log statements, and in a distributed system, multiple *processes*. A **correlation ID** (also called a request ID or trace ID) is a unique identifier generated once per incoming request and threaded through every log line produced while handling it, so you can filter your log aggregator by that one ID and reconstruct the entire request's story in order.

Implementation pattern for FastAPI: a middleware generates (or extracts, if the client/load balancer already supplied one via an `X-Request-ID` header) a UUID at the start of each request, stores it in a `contextvars.ContextVar` (so it's available anywhere in the async call stack without threading it through every function signature), and a `structlog` processor reads that context var and injects it into every log entry automatically:

```python
import contextvars
import uuid
from fastapi import Request

correlation_id_var = contextvars.ContextVar("correlation_id", default="-")

@app.middleware("http")
async def correlation_id_middleware(request: Request, call_next):
    cid = request.headers.get("X-Request-ID", str(uuid.uuid4()))
    correlation_id_var.set(cid)
    response = await call_next(request)
    response.headers["X-Request-ID"] = cid
    return response
```

Pitfall: using a plain global variable instead of `contextvars` — under `asyncio`, concurrently-handled requests would then stomp on each other's correlation ID, since a normal module-level variable is shared across all coroutines running on the same event loop.

---

## Application monitoring

### Metrics (Prometheus-style)

Logs answer "what happened in this one request." **Metrics** answer aggregate questions over time — "what's our p99 latency for the last hour," "how many requests per second are we serving," "how many tasks are currently in the 'overdue' state." Prometheus popularized a simple, widely-adopted model: an application exposes a `/metrics` HTTP endpoint returning current counter/gauge/histogram values in a plain-text format, and a Prometheus server periodically scrapes (polls) that endpoint and stores the time series.

Three core metric types matter most:
- **Counter** — a value that only increases (total requests served, total tasks created). Useful for rate calculations (`rate(requests_total[5m])`).
- **Gauge** — a value that goes up and down (current number of open DB connections, current queue depth).
- **Histogram** — buckets observations (e.g., request duration) so you can compute percentiles (p50, p95, p99) after the fact.

For FastAPI, the `prometheus-fastapi-instrumentator` package (or hand-rolled middleware using `prometheus_client`) can automatically track request count and latency histograms per route and status code.

Pitfall: treating average latency as sufficient. Averages hide the tail — a service with a 50ms average but a 3-second p99 is failing 1% of users badly, and averages will never reveal that; always look at percentiles for latency.

### Health checks and uptime checks

A **health check** endpoint (conventionally `/health` or `/healthz`) tells an orchestrator (a load balancer, Kubernetes, or a PaaS) whether the instance is ready to serve traffic. A good health check actually verifies dependencies are reachable (can we reach the database? Redis?) rather than just returning `200 OK` unconditionally — an unconditional "always healthy" check is worse than no check at all, because it hides real outages from your infrastructure's automatic recovery mechanisms (e.g., restarting a broken container). At the same time, a health check shouldn't be *too* strict (failing on a single transient Redis blip) or it causes unnecessary restarts — this distinction is often split into "liveness" (is the process alive at all) vs "readiness" (is it ready to serve traffic) checks.

An **uptime check** is external: a third-party service (or a simple cron job) periodically hits your public URL from outside your infrastructure and alerts you if it doesn't respond — this catches failures your own health check can't, like DNS misconfiguration or the entire host being unreachable.

---

## Error tracking

Logs are for events; an **error tracker** (Sentry is the standard example) is specifically for exceptions — it captures the full stack trace, the request context, deduplicates identical errors into one "issue" instead of flooding you with duplicates, tracks how many users/requests are affected, and can alert you the first time a *new* kind of error appears. Integrating Sentry into FastAPI is typically a few lines (`sentry_sdk.init(dsn=..., integrations=[FastApiIntegration()])`), after which unhandled exceptions are automatically captured with request context attached.

The key difference from plain logging: Sentry groups occurrences of the *same* underlying bug together (even across thousands of requests) and shows you a trend line, whereas grep-ing logs for `Traceback` gives you a flat, undeduplicated list you have to manually cluster yourself.

---

## Debugging production issues without a debugger

You cannot attach `pdb` to a production process serving live traffic (and even if you technically could, you shouldn't — it blocks the process). Production debugging is almost entirely **log analysis and request tracing**: given a correlation ID from a user's bug report or an alert, you search your log aggregator for every log line carrying that ID, reconstruct the sequence of events, and look for the first anomaly (an unexpected branch taken, a slow downstream call, an exception).

This is why structured logging with correlation IDs (above) isn't optional polish — it is the entire mechanism by which production issues get diagnosed. A service with only unstructured `print()`-style logging and no request correlation is effectively undebuggable once more than one request is in flight at a time.

---

## Performance profiling

### `cProfile`

`cProfile` is a deterministic, built-in profiler that instruments every function call and reports how much time was spent in each — total time, cumulative time (including callees), and call counts. Run it as `python -m cProfile -o output.prof my_script.py` and inspect with `pstats` or visualize with `snakeviz`. It's precise but has real overhead (every call is instrumented), so it's a *development-time* diagnostic tool, not something you leave running in production.

```python
import cProfile
import pstats

profiler = cProfile.Profile()
profiler.enable()
run_slow_operation()
profiler.disable()
stats = pstats.Stats(profiler).sort_stats("cumulative")
stats.print_stats(15)
```

### `py-spy`

`py-spy` is a **sampling** profiler that attaches to an already-running Python process from *outside* it (no code changes, no restart) and periodically samples the call stack. Because it samples rather than instruments every call, overhead is far lower — low enough that it's commonly used to profile live production processes for a short window (`py-spy record -o profile.svg --pid 1234` produces a flame graph). This is the tool you reach for when you can't (or don't want to) modify and redeploy code just to diagnose a slowdown.

The complementary use of the two: use `cProfile` locally against a reproducible slow case to get exact call counts and cumulative time; use `py-spy` against the actual running production process when you can't reproduce the slowness locally, or when you need zero-overhead observation of live traffic.

### Memory profiling: `tracemalloc` and `memory_profiler`

`tracemalloc` (stdlib) tracks where each allocation in your Python code happened, letting you snapshot memory usage and diff two snapshots to find what grew — invaluable for finding memory leaks (e.g., a cache that never evicts, or a list that accumulates across requests). `memory_profiler` (third-party) gives line-by-line memory usage for a decorated function, which is more granular but has higher overhead and is typically used in a focused, local investigation rather than continuously.

Pitfall: assuming Python's garbage collector immediately frees objects the moment they go out of scope — reference cycles, module-level caches, and objects held by a still-open closure often keep memory alive far longer than intuition suggests; `tracemalloc` snapshots are how you find that empirically instead of guessing.

---

## Basic distributed tracing concepts

Once a system has more than one service talking to each other (e.g., TaskFlow calling out to a notification service, or a future microservices split), a single correlation ID tells you *that* a request touched multiple logs, but not the timing relationship *between* services — how long was spent in service A versus waiting on service B. **Distributed tracing** (OpenTelemetry is the current standard) extends the correlation ID idea into a "trace" made of nested "spans" — each span records a start time, duration, and parent span, so a single request across multiple services can be visualized as a waterfall diagram showing exactly where time went. For a single-service app like TaskFlow this is optional, but understanding the concept now means the mental model is already in place before Chapter 12's microservices track.
