# Chapter 9 Project — Dockerize and Ship TaskFlow

## Where TaskFlow stands entering Chapter 9

By the end of Chapter 8, TaskFlow is a FastAPI REST API with JWT auth, a layered architecture (domain / service / repository / API), SQLAlchemy + SQLite persistence, a Redis cache layer for read-heavy endpoints, async support on IO-heavy paths, and a full typed pytest suite with linting/type-checking enforced via pre-commit. It runs, but only ever on a developer's own machine via `uvicorn` and a manually-started Redis instance. There is no automated pipeline and no way for anyone else to run it without replicating your local setup by hand.

## What to add this chapter

1. **Multi-stage Dockerfile.** Add a `Dockerfile` at the repo root:
   - Builder stage: `python:3.12-slim`, installs build dependencies, installs TaskFlow's dependencies from `pyproject.toml`/lock file into an isolated location.
   - Final stage: fresh `python:3.12-slim`, copies only the installed packages and `src/`, creates and switches to a non-root user, sets `PYTHONUNBUFFERED=1`, exposes port 8000, and runs `uvicorn taskflow.api.main:app --host 0.0.0.0 --port 8000`.
   - Add a `.dockerignore` excluding `.venv`, `__pycache__`, `.git`, `.env`, `tests/`, local `*.db` files, and coverage/mypy caches.

2. **docker-compose.yml for local multi-service dev.** Define three services:
   - `app` — builds from the local Dockerfile, mounts `./src` for live editing, depends on `db` and `cache`.
   - `db` — Postgres (this chapter is the natural point to move off SQLite for anything beyond pure local dev; keep SQLite as the default for the test suite, but demonstrate Postgres via Compose), with a healthcheck.
   - `cache` — Redis, matching the existing cache layer from Chapter 5.
   - Wire `DATABASE_URL` and `REDIS_URL` into the `app` service as environment variables consumed by the existing `pydantic-settings` config object — no code changes required to point at Compose services versus local SQLite/Redis.

3. **GitHub Actions CI/CD pipeline** at `.github/workflows/ci.yml`:
   - `lint-and-type-check` job: `ruff check` and `mypy` on every push and PR.
   - `test` job (needs `lint-and-type-check`): install with dev extras, run `pytest` with coverage, against SQLite (no external services needed for the unit/integration suite).
   - `build-image` job (needs `test`, gated to `push` on `main` only): builds the Docker image and pushes it to a registry (GitHub Container Registry, `ghcr.io`, is the simplest — it needs no extra account setup beyond the repo's own `GITHUB_TOKEN`).

4. **Environment parity.** Confirm the same Docker image can run against local SQLite/Redis (dev), a Compose-provided Postgres/Redis (local "staging-like" testing), and a hosted Postgres/Redis (production), purely by changing environment variables — never by branching in code or building a different image per environment.

5. **Deploy to a free-tier host.** Connect the GitHub repo to a PaaS (Render, Railway, or Fly.io), configure it to build from the Dockerfile, set `DATABASE_URL`, `REDIS_URL`, and `JWT_SECRET` as platform-managed secrets (never committed), and deploy. Confirm the deployed API is reachable from the public internet (e.g., hitting `/health` or the OpenAPI docs URL) and that a fresh `git push` to `main` triggers an automatic redeploy.

6. **Secrets hygiene.** Ensure `.env` is git-ignored, add a `.env.example` documenting required variable names with empty/placeholder values, and confirm no real secret ever appears in a commit, a Dockerfile `ENV` instruction, or a CI log.

## Definition of done

- [ ] `docker build .` succeeds locally and produces an image running as a non-root user.
- [ ] `docker compose up` brings up `app` + `db` + `cache` together, and the API is reachable at `localhost:8000` with data persisting across container restarts (named volume).
- [ ] `.dockerignore` prevents `.env`, `.git`, and virtual environment files from ever entering the build context.
- [ ] GitHub Actions runs lint + type-check + tests on every push to any branch, and additionally builds and pushes a Docker image only on merge to `main`.
- [ ] The same image/config mechanism works unmodified against local SQLite/Redis and a Compose-provided Postgres/Redis, driven only by environment variables.
- [ ] TaskFlow is deployed to a free-tier PaaS and answers requests from the public internet.
- [ ] No secret exists in git history, Dockerfile, or CI logs; a `.env.example` documents required configuration.
- [ ] A fresh `git clone` of the repo, following only the README, can reach a running local instance via `docker compose up` with zero undocumented manual steps.
