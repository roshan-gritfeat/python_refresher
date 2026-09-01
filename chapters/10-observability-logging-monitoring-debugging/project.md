# Chapter 10 Project — Make TaskFlow Observable

## Where TaskFlow stands entering Chapter 10

TaskFlow is now Dockerized, has a docker-compose stack (app + Postgres + Redis), a GitHub Actions pipeline (lint, type-check, test, build-on-merge), and is deployed to a free-tier PaaS from Chapter 9. It works, but if a user reports "the app is slow" or "my request failed," there is currently no way to investigate beyond re-reading source code and guessing — no structured logs, no metrics, no health check, no profiling evidence.

## What to add this chapter

1. **Structured logging with correlation IDs across the request lifecycle.**
   - Introduce `structlog` (or a stdlib `logging` + JSON formatter setup) configured once at app startup.
   - Add a FastAPI middleware that assigns a correlation ID (from `X-Request-ID` if the client sent one, otherwise a generated UUID) per request, stored in a `contextvars.ContextVar`, echoed back in the response header.
   - Ensure every log statement across the API, service, and repository layers automatically includes the correlation ID (via a `structlog` processor reading the context var) — no need to manually pass it into every function signature.
   - Log meaningful business events (task created, task completed, auth failure) at `INFO`, and unexpected exceptions at `ERROR`, with consistent field names (`event`, `user_id`, `task_id`, etc.).

2. **Health-check and metrics endpoints.**
   - Add `GET /health` that checks the database connection and Redis connection are reachable, returning `200` when healthy and `503` with details on which dependency failed otherwise.
   - Add `GET /metrics` exposing Prometheus-format counters (total requests by route/status) and a histogram of request latency, using `prometheus_client` or `prometheus-fastapi-instrumentator`.

3. **Deliberately introduce and then fix a performance bottleneck.**
   - Introduce a realistic bottleneck into one read-heavy endpoint (e.g., an N+1-style query re-fetch inside a loop, or a synchronous blocking call left inside an otherwise async path) on a throwaway branch.
   - Reproduce the slowdown locally and profile it with `cProfile`, identifying the specific hot function from the `pstats` output.
   - Separately, run the running dev server under `py-spy` to show the same bottleneck visible via sampling profiling without code instrumentation.
   - Fix the bottleneck (e.g., batch the query, make the call properly async/non-blocking) and re-profile to demonstrate and quantify the improvement.
   - Keep a short before/after note (in the branch's PR description or a `PROFILING.md`) documenting the bottleneck, the diagnosis method, and the fix — this is the artifact that proves the "tool-driven process" the syllabus asks for.

4. **Wire logs and correlation IDs into error tracking (optional but recommended).** If time permits, connect a free-tier Sentry project so unhandled exceptions in production are captured with the correlation ID attached as extra context.

## Definition of done

- [ ] Every request produces structured (JSON) log lines carrying a shared correlation ID across API, service, and repository layers.
- [ ] `X-Request-ID` is honored if supplied by the client, generated otherwise, and returned in the response headers.
- [ ] `GET /health` verifies real dependencies (DB, Redis) and returns `503` when one is down, not just an unconditional `200`.
- [ ] `GET /metrics` exposes request counters and a latency histogram in Prometheus text format.
- [ ] A deliberately introduced performance bottleneck was reproduced, profiled with both `cProfile` and `py-spy`, fixed, and the improvement is quantified (before/after timing).
- [ ] Given only a correlation ID, you can reconstruct the full story of a single request from the logs alone.
- [ ] README (or a `PROFILING.md`) documents the bottleneck investigation as a reusable process for future incidents.
