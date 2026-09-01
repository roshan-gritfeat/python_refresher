# Chapter 12 — Practice Problems

Work through the problems for whichever 2–3 tracks you chose in the README. Each problem is meant to be a focused drill (a few hours, not a multi-day build) — save the full stretch feature for `project.md`.

---

## Track: Data Engineering

1. **Pandas → Polars migration.** Take a CSV of at least 500k rows (synthetic is fine — generate one with `faker` or `numpy.random`) and write the same groupby-aggregate-sort pipeline twice: once in `pandas`, once in `polars` (using its lazy API with `.collect()` at the end). Benchmark both with `time` or `timeit` and explain the difference in your own words.
2. **Idempotent incremental load.** Write a small ETL script that reads new rows from a source table (simulate with a `last_updated` timestamp column), transforms them, and loads them into a destination table — such that running the script twice in a row does not create duplicates.
3. **Data validation gate.** Using `pandera` (or `great_expectations`), define a schema for a "task export" dataset (columns: `id`, `title`, `status`, `created_at`, `assignee_email`). Feed it a deliberately corrupted dataset (nulls where they shouldn't be, an out-of-range status, a malformed email) and confirm validation fails with a clear, actionable error.
4. **Orchestrate a 3-step DAG.** Using `Prefect` (or `Airflow` if you prefer the heavier setup), build a 3-task pipeline (extract → transform → load) with an explicit retry policy on the extract step, and demonstrate a failure in step 2 does not silently continue to step 3.

---

## Track: Microservices / Backend

1. **Broker comparison, hands-on.** Stand up RabbitMQ (Docker) and publish/consume a simple "hello" message using `pika` or `aio-pika`. Then do the same with Kafka (Docker, e.g. `confluent-kafka` or `aiokafka`). Write down, from direct experience, one concrete difference you noticed in setup, delivery guarantees, or consumer behavior.
2. **At-least-once delivery.** Build a producer/consumer pair where the consumer can crash mid-processing and, on restart, does not lose the in-flight message — but may process it twice. Then add idempotency handling (e.g., a processed-message-ID set or unique constraint) so duplicate processing is safe.
3. **Service-to-service auth.** Build two toy FastAPI services where Service A calls a protected endpoint on Service B. Implement short-lived JWT-based service auth (A obtains a token, B validates it) and write a test proving Service B rejects an expired or tampered token.
4. **Minimal API gateway.** Build a small gateway (a FastAPI app or `nginx`/`traefik` config) that routes `/orders/*` and `/users/*` to two different backend services, and adds a single shared rate limit across both. Explain what breaks if the gateway itself goes down, and one mitigation.

---

## Track: ML Engineering

1. **Serve a trained model.** Train a trivial scikit-learn model (e.g., logistic regression on a small tabular dataset) and serve it behind a FastAPI endpoint with Pydantic input validation. Ensure the model loads once at startup, not per-request — prove this with a log line or timer.
2. **Model versioning.** Save two versions of a model artifact (`v1`, `v2` — even just retrained with different hyperparameters) and add a `?model_version=` query param (or header) to your serving endpoint that lets a caller pin a version. Default to the latest.
3. **Experiment tracking.** Use MLflow (or a simple CSV/JSON log if you want zero infra) to record hyperparameters and evaluation metrics across 3+ training runs of the same model with different settings, and identify the best run programmatically (not by eyeballing).
4. **Drift detection sketch.** Write a function that compares the distribution of a numeric input feature between training data and a batch of live prediction requests (e.g., using a simple statistical test or just a mean/std threshold) and flags when they diverge meaningfully.

---

## Track: High-performance Python

1. **Profile before you optimize.** Take a deliberately slow pure-Python function (e.g., computing pairwise distances between points in nested loops) and profile it with `cProfile`. Identify the actual hot line, not the one that looks slow.
2. **Vectorize it.** Rewrite the function from problem 1 using `numpy` vectorized operations. Benchmark the speedup and explain, precisely, why it's faster (not just "numpy is fast").
3. **Cythonize a hot loop.** Take a numeric function that doesn't vectorize cleanly (e.g., has data-dependent branching per element) and write a Cython version with static type annotations. Benchmark against the pure-Python version.
4. **`py-spy` on a running process.** Start a long-running Python script with an artificial bottleneck, attach `py-spy` to the live process (no restart, no code changes) and produce a flame graph. Identify the bottleneck from the flame graph alone.

---

## Track: DevOps / Platform

1. **First Kubernetes deployment.** Write a `Deployment` and `Service` manifest for a simple containerized app (can be a "hello world" FastAPI app) and deploy it to a local cluster (`minikube` or `kind`). Confirm it's reachable and that killing a pod causes Kubernetes to restart it automatically.
2. **Readiness vs liveness.** Add both a readiness and a liveness probe to your deployment, with different behavior (e.g., readiness fails while a slow startup task runs; liveness fails on a deliberately triggered deadlock). Observe how Kubernetes treats each differently.
3. **Terraform a real resource.** Use Terraform to provision one real piece of cloud infrastructure (e.g., an S3/GCS bucket, or a managed Postgres instance on a free tier). Run `terraform plan`, apply it, then destroy it — and explain what the state file is for.
4. **Blue-green or canary deploy.** Simulate a blue-green deployment locally (two versions of an app running behind a router/proxy you control) and write the steps to cut traffic over from blue to green, plus the rollback steps if green is unhealthy.
