# Chapter 12 — Project: Extend TaskFlow with a Track-Specific Stretch Feature

## Do This

Pick **one** of the five tracks below and extend TaskFlow with the corresponding stretch feature, end-to-end. Don't try to do all five — the point is depth in one direction, not breadth across all of them. Each option describes a concrete feature, what it touches in the existing TaskFlow architecture, and a definition-of-done checklist.

By now TaskFlow has: a layered architecture (domain / service / repository / API), async FastAPI endpoints, SQLAlchemy + Alembic migrations, a full test suite, CI/CD, structured logging, health/metrics endpoints, and security hardening. Whichever option you pick should plug into that architecture, not bypass it.

---

## Option A — Data Engineering: TaskFlow Analytics ETL

Build a pandas/Airflow (or Prefect) ETL pipeline that exports TaskFlow's task data into an analytics-friendly form — e.g., a daily "tasks created / completed / overdue by team" rollup written to a separate reporting table or a Parquet file.

**Touches:** a new read-only path against the existing repository layer (or a direct read replica query — don't let the ETL job write through your API), a new `analytics/` module or standalone pipeline project, an orchestration DAG.

**Definition of done:**
- [ ] Pipeline extracts task data from TaskFlow's database without going through the REST API (direct read against a replica or read-only credentials)
- [ ] Transform step produces at least one meaningful aggregation (e.g., completion rate per team per day)
- [ ] Load step is idempotent — running the pipeline twice for the same day does not duplicate rows
- [ ] Data validation step (pandera or great_expectations) rejects malformed extracts before they reach the load step
- [ ] Orchestrated with Airflow or Prefect, with retries configured on the extract step
- [ ] A README section explaining how to run the pipeline locally and what the output looks like

---

## Option B — Microservices/Backend: Event Pipeline for Task Lifecycle Events

Add a Kafka or RabbitMQ event pipeline that publishes events (`task.created`, `task.completed`, `task.assigned`) whenever the corresponding domain action happens in TaskFlow, and build at least one independent consumer that reacts to them (e.g., a notification service that logs/emails on `task.assigned`).

**Touches:** the service layer (publish events after a successful domain operation, not before — don't publish on a failed transaction), a new lightweight consumer service, infrastructure config (Docker Compose service for the broker).

**Definition of done:**
- [ ] Events are published only after the originating DB transaction commits successfully (no false events on rollback)
- [ ] Event schema is explicit and versioned (e.g., a Pydantic model or Avro/JSON schema with a `version` field)
- [ ] At least one independent consumer service processes events asynchronously from the main API
- [ ] Consumer handles duplicate delivery safely (idempotent processing)
- [ ] Broker (Kafka or RabbitMQ) added to `docker-compose.yml` so local dev spins up the full pipeline with one command
- [ ] A test proving an event is published when a task is created, using a test double or embedded broker

---

## Option C — ML Engineering: Task Priority Predictor

Train a small ML model that predicts a task's priority (e.g., low/medium/high/urgent) from its title, description, and metadata, and serve it via a new FastAPI endpoint inside (or alongside) TaskFlow.

**Touches:** a new `ml/` module for training code, a new API route (e.g., `POST /tasks/{id}/predict-priority`) in the existing API layer, a model artifact loaded once at app startup.

**Definition of done:**
- [ ] Training script produces a versioned model artifact from a labeled dataset (synthetic labels are fine if no real historical data exists)
- [ ] Model is loaded once at application startup, not per-request
- [ ] New endpoint validates input with Pydantic and returns a prediction plus a confidence score
- [ ] Endpoint is covered by the existing test suite conventions (unit test with a mocked/stubbed model, at least one integration test)
- [ ] Basic model metadata (version, trained-on date) is exposed via the endpoint or a companion `/model-info` route
- [ ] A README section explaining how the model was trained and how to retrain it

---

## Option D — High-performance: Vectorized Bulk Import

TaskFlow currently creates tasks one at a time through the API. Add a bulk-import path (e.g., `POST /tasks/bulk-import` accepting a CSV/JSON of thousands of tasks) that uses numpy vectorization (and optionally a Cython-optimized validation/dedup step) to process the batch far faster than a naive per-row loop.

**Touches:** a new service-layer bulk-import function, a new API route, a benchmark script comparing naive vs optimized paths.

**Definition of done:**
- [ ] Naive (row-by-row Python loop) implementation exists first, and is profiled with `cProfile` to establish a baseline
- [ ] Optimized implementation uses numpy vectorization for at least the validation/dedup/transform step (and Cython for any step that doesn't vectorize cleanly)
- [ ] Benchmark script demonstrates and reports the measured speedup on a realistic batch size (e.g., 50k rows)
- [ ] Optimized path still enforces the same validation rules as the single-task creation path (no correctness regressions for speed)
- [ ] Bulk import is transactional — a failure partway through does not leave a half-imported batch (or partial success is explicitly reported, not silently dropped)
- [ ] A test suite covering both correctness and a duplicate-detection case

---

## Option E — DevOps/Platform: Kubernetes Deployment

Write a production-style Kubernetes deployment for TaskFlow: Deployment, Service, ConfigMap/Secret, readiness/liveness probes, and horizontal pod autoscaling — replacing (or sitting alongside) the current free-tier PaaS deployment from Chapter 9.

**Touches:** new `k8s/` manifests (or a Helm chart), CI/CD pipeline updated to apply manifests or push a Helm release, no changes needed to application code beyond exposing the existing health/metrics endpoints correctly.

**Definition of done:**
- [ ] Deployment manifest runs TaskFlow's existing Docker image with correct resource requests/limits
- [ ] Readiness probe uses the existing health-check endpoint; liveness probe is distinct and catches a genuinely hung process
- [ ] Secrets (DB credentials, JWT signing key, etc.) are injected via Kubernetes Secrets, never baked into the image or manifest in plaintext
- [ ] A Service (and optionally Ingress) exposes the app, verified reachable from outside the cluster
- [ ] HorizontalPodAutoscaler configured on a real metric (CPU or request rate) with a test that generates enough load to trigger a scale-up
- [ ] CI/CD pipeline updated to deploy to the cluster on merge to main, with a documented rollback step

---

## You'll know you're done when...

You can walk someone through your chosen track's addition end-to-end — what it does, why you built it this way, what would break it, and how you'd know if it broke in production — for long enough that it clearly isn't a copy-pasted tutorial exercise.
