# Chapter 9 — DevOps Essentials for Python Developers

**Goal:** "works on my machine" stops being acceptable. This chapter turns TaskFlow from a project you run locally into a service that can be built, tested, and deployed reproducibly by anyone (including a CI runner that has never seen your laptop).

---

## Docker

### Writing efficient Dockerfiles for Python apps

A Dockerfile is a recipe for building an image — a self-contained filesystem snapshot that includes your interpreter, your dependencies, and your code. The point isn't just "it runs in a container"; it's *reproducibility*: the exact same image runs identically on your laptop, in CI, and in production, because the OS packages, Python version, and library versions are frozen at build time instead of drifting per-machine.

Key practices for Python images:

- **Pin your base image tag.** `FROM python:3.12-slim` is better than `FROM python:latest` — `latest` silently changes over time and breaks reproducibility. `-slim` variants trade a smaller image for having to `apt-get install` build tools yourself when a dependency needs compilation.
- **Leverage layer caching.** Docker caches each instruction as a layer and reuses it if nothing upstream changed. Copy `pyproject.toml`/`requirements.txt` and install dependencies *before* copying your application code, so that editing a `.py` file doesn't force a full dependency reinstall on every build.
- **Don't run as root.** Create a non-privileged user and switch to it with `USER appuser` before the final `CMD`. Running production containers as root is a common, easily-avoided attack-surface increase.
- **Set `PYTHONDONTWRITEBYTECODE=1` and `PYTHONUNBUFFERED=1`** as environment variables — the first avoids littering the image with `.pyc` files, the second ensures `print`/logging output isn't buffered and lost when a container is killed.

Common pitfall: copying the entire repository (`COPY . .`) before installing dependencies. This busts the cache on every single code change, making builds slow. Also common: forgetting to remove build-only dependencies (compilers, dev headers) from the final image, bloating it unnecessarily — this is exactly what multi-stage builds solve.

### Multi-stage builds

A multi-stage build uses multiple `FROM` statements in one Dockerfile. Earlier stages can install compilers and build wheels; the *final* stage copies only the compiled artifacts (e.g., installed site-packages) into a clean, minimal base image. The build tools, source tarballs, and caches from the builder stage never end up in the shipped image.

```dockerfile
# ---- builder stage ----
FROM python:3.12-slim AS builder
WORKDIR /app
RUN apt-get update && apt-get install -y --no-install-recommends build-essential \
    && rm -rf /var/lib/apt/lists/*
COPY pyproject.toml poetry.lock ./
RUN pip install --no-cache-dir --target=/install .

# ---- final stage ----
FROM python:3.12-slim
WORKDIR /app
RUN useradd -m appuser
COPY --from=builder /install /usr/local/lib/python3.12/site-packages
COPY ./src ./src
USER appuser
ENV PYTHONUNBUFFERED=1
EXPOSE 8000
CMD ["uvicorn", "taskflow.api.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Why this matters for TaskFlow specifically: TaskFlow depends on things like `cryptography` or database drivers that sometimes need C compilation. Without multi-stage builds, your production image would permanently carry `gcc`, `make`, and header files it never needs again after the wheel is built — larger images mean slower deploys, larger attack surface, and slower cold starts on platforms that pull images on demand.

### `.dockerignore`

Just like `.gitignore`, `.dockerignore` excludes files from the build context sent to the Docker daemon. Without it, `COPY . .` will happily copy `.venv/`, `__pycache__/`, `.git/`, test artifacts, and local `.env` files into your image — bloating it and, worse, potentially leaking secrets or credentials into a shipped artifact. A minimal one for TaskFlow:

```
.git
.venv
__pycache__
*.pyc
.pytest_cache
.mypy_cache
.env
tests/
docs/
*.db
```

Pitfall: forgetting `.dockerignore` means every local change to `__pycache__` or `.pytest_cache` invalidates Docker's cache and re-triggers unnecessary layer rebuilds, even though nothing meaningful changed.

---

## Docker Compose for local multi-service dev

TaskFlow, by Chapter 8, is a FastAPI app backed by SQLAlchemy/SQLite and Redis. In production you'd likely swap SQLite for Postgres, and locally you want the app, database, and cache running together with one command instead of juggling three terminals. `docker-compose.yml` declares all services, their images, ports, volumes, and how they depend on each other:

```yaml
services:
  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql+asyncpg://taskflow:taskflow@db:5432/taskflow
      - REDIS_URL=redis://cache:6379/0
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    volumes:
      - ./src:/app/src   # live-reload in dev

  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=taskflow
      - POSTGRES_PASSWORD=taskflow
      - POSTGRES_DB=taskflow
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U taskflow"]
      interval: 5s
      timeout: 3s
      retries: 5
    volumes:
      - pgdata:/var/lib/postgresql/data

  cache:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  pgdata:
```

Key concepts: **service names become DNS hostnames** inside the Compose network (`db`, `cache`), so your app's connection strings use the service name, not `localhost`. **`depends_on` with `condition: service_healthy`** ensures the app doesn't start racing against a database that hasn't finished initializing — a very common source of flaky "connection refused" errors on first `docker compose up`. **Named volumes** (`pgdata`) persist database state across container restarts, so you don't lose your local data every time you rebuild.

Pitfall: relying on `depends_on` alone (without a healthcheck) only waits for the container process to *start*, not for Postgres to actually be ready to accept connections — a classic cause of intermittent startup failures.

---

## CI/CD pipelines

### GitHub Actions — lint, test, build, deploy stages

CI/CD automates the steps that used to be manual and error-prone: "did you remember to run the tests before merging?" A pipeline is a declarative description of jobs that run on every push or on specific events (like a merge to `main`).

The philosophy: **fail fast, fail cheap.** Linting and type-checking are near-instant and should run before the (slower) test suite. Building a Docker image is the most expensive step, so it should only happen once code has already passed cheaper gates, and ideally only on `main` (not every feature-branch push) to avoid wasting CI minutes.

A typical structure for TaskFlow:

```yaml
name: CI
on:
  push:
    branches: ["**"]
  pull_request:

jobs:
  lint-and-type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install ruff mypy
      - run: ruff check .
      - run: mypy src

  test:
    needs: lint-and-type-check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install -e ".[dev]"
      - run: pytest --cov=taskflow --cov-report=xml

  build-image:
    needs: test
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
```

Notice the `needs:` chain — each job only runs if the previous one succeeded, and `build-image` is additionally gated on branch and event so pull requests never push images. This mirrors exactly what the syllabus asks for: lint + type-check + test on every push, image build only on merge to `main`.

### Environment parity: dev/staging/prod configuration

"Environment parity" means dev, staging, and production should differ as little as possible — same container image, same dependency versions, same OS — with the *only* differences being configuration values (database URLs, feature flags, log levels) injected at runtime, not baked into the image or the code. TaskFlow already uses `pydantic-settings` from Chapter 2 for this; Chapter 9 extends it so the same image can be pointed at a local SQLite/Redis pair, a staging Postgres instance, or a production one purely through environment variables — never through an `if ENV == "prod"` branch that changes behavior.

Pitfall: building a *different* image per environment (e.g., a "staging Dockerfile" and a "prod Dockerfile"). This defeats the purpose — you're no longer testing the artifact you actually ship. Build once, promote the same image through environments.

---

## Basic cloud deployment

Once you have a container image, you need somewhere to run it. Two broad options:

- **PaaS (Platform-as-a-Service)** — Render, Railway, Fly.io. You point them at your Dockerfile or a `git` repo, they handle networking, TLS certificates, and scaling for you. This is the right starting point for a solo/self-study project: minimal YAML, fast to set up, generous free tiers.
- **Cloud-native container platforms** — AWS ECS/Fargate, GCP Cloud Run. More powerful and more "real world enterprise," but require understanding IAM roles, VPCs, load balancers, and task definitions — a much steeper learning curve for a first deployment.

For TaskFlow, the pragmatic choice is a PaaS like **Render** or **Fly.io**: connect the GitHub repo, point it at the Dockerfile, set the environment variables (`DATABASE_URL`, `REDIS_URL`, `JWT_SECRET`) in the platform's dashboard (never in code), and let it build and deploy automatically on push to `main`. Fly.io additionally offers a managed Postgres and Redis (Upstash) add-on, so the whole stack can live on free/hobby tiers.

Common pitfall: forgetting that most PaaS free tiers spin containers down when idle, causing a slow "cold start" on the first request after inactivity — worth knowing so you don't mistake it for a bug when benchmarking.

---

## Secrets management basics

The cardinal rule: **secrets never get committed to the repository**, not even for "temporary testing." Once a secret is in git history, rotating it is the only real fix — deleting the file in a later commit doesn't remove it from history.

Practical approach for TaskFlow:

- Keep a `.env.example` file in the repo with variable *names* but no real values (`JWT_SECRET=`, `DATABASE_URL=`), and add `.env` to `.gitignore`.
- In CI, store real secrets as **GitHub Actions encrypted secrets** (`Settings → Secrets and variables → Actions`), referenced as `${{ secrets.JWT_SECRET }}` — GitHub redacts them from logs automatically.
- In production, inject secrets as environment variables through the hosting platform's secret manager (Render/Fly.io dashboards, AWS Secrets Manager, etc.), not baked into the Docker image — an image is often more widely distributed/cached than you'd expect, and baked-in secrets travel with it.

Pitfall: logging the full environment or a config object at startup "for debugging" — this is one of the most common accidental secret leaks, since log aggregators often have far more people with read access than the production database itself.
