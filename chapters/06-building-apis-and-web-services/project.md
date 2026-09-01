# Chapter 6 Project — TaskFlow: Building APIs & Web Services

## Where TaskFlow stands entering this chapter

By the end of Chapter 5, TaskFlow is an installable Python package with:
- A `src`-layout package, tested with `pytest`, linted/typed with `ruff`/`mypy`, and enforced via `pre-commit`.
- A SQLAlchemy ORM persistence layer backed by SQLite, with Alembic migrations.
- A Redis-backed cache layer on at least one read-heavy operation.
- No externally reachable interface yet — everything is exercised through Python function calls and tests, not over the network.

## What to add this chapter

Expose TaskFlow's task-tracking logic as a REST API, so it can be used by a frontend or any HTTP client, with authentication protecting the endpoints that need it.

### 1. Stand up a FastAPI application

- Add `fastapi` and `uvicorn` (and `python-jose` or `pyjwt` for JWTs) to `pyproject.toml` dependencies.
- Create an `api` module inside the package (e.g., `taskflow/api/`) containing the FastAPI `app` instance, keeping it separate from the existing DB/service code — don't collapse everything into one file.
- Add a `console_scripts` entry point or a documented `uvicorn taskflow.api.main:app` command to run the server locally.

### 2. Model requests and responses with Pydantic

- Define `TaskCreate`, `TaskUpdate`, and `TaskRead` Pydantic models — distinct from the SQLAlchemy `Task` model from Chapter 5. `TaskRead` should never accidentally expose internal-only columns.
- Wire up endpoints backed by the existing DB layer:
  - `POST /tasks` — create a task (`201`, with `Location` header).
  - `GET /tasks` — list tasks, with pagination (`offset`/`limit`) and at least one filter (e.g., `status`).
  - `GET /tasks/{id}` — fetch one (`404` if missing).
  - `PATCH /tasks/{id}` — partial update (idempotent).
  - `DELETE /tasks/{id}` — remove (`204`).
- Use FastAPI's dependency injection (`Depends`) to provide a DB session per request, reusing the session/engine setup from Chapter 5 rather than duplicating it.

### 3. Add JWT authentication

- Add `POST /auth/register` and `POST /auth/login` endpoints; store hashed passwords (bcrypt/argon2 — a full deep-dive comes in Chapter 11, but don't store plaintext passwords now).
- Issue a signed JWT on successful login with a reasonable expiry.
- Protect the task-mutating endpoints (`POST`, `PATCH`, `DELETE /tasks/...`) with a dependency that validates the `Authorization: Bearer` header and identifies the current user; scope tasks to their owner so one user cannot modify another's tasks.
- Leave `GET /tasks` for the authenticated user's own tasks only (no cross-user leakage).

### 4. Consistent error handling

- Add a global exception handler translating your domain exceptions (e.g., `TaskNotFoundError`, `NotAuthorizedError`) and validation failures into one consistent JSON error shape.
- Make sure unhandled exceptions never leak a stack trace to the client in the API response.

### 5. Review and fix the generated OpenAPI docs

- Run the app and open `/docs`. Check every endpoint for: correct response models, documented error responses (`404`, `401`, `403`, `422`), and meaningful examples.
- Fix at least two things that were misleading or missing by default (e.g., an endpoint returning `dict` instead of a typed model, a missing description on a field, an undocumented error code).

### 6. Cover it with tests

- Add API-level tests using `fastapi.testclient.TestClient` (or `httpx.AsyncClient`), covering: successful CRUD flows, auth failures (missing/invalid/expired token), ownership enforcement (user A cannot touch user B's tasks), and pagination/filter edge cases.
- Keep these alongside the existing unit tests from Chapter 3, run through the same `pytest` + coverage setup.

## Definition of done

- [ ] FastAPI app runs locally and serves `/tasks` CRUD backed by the Chapter 5 SQLAlchemy layer.
- [ ] Pydantic request/response models exist and are distinct from persistence models (no internal fields leaked).
- [ ] `POST /auth/register` and `POST /auth/login` work; passwords are hashed, never stored in plaintext.
- [ ] Protected endpoints reject missing/invalid/expired JWTs with `401`, and reject cross-user access with `403`/`404` as appropriate.
- [ ] All endpoints return correct, consistent status codes and a uniform error response shape.
- [ ] Pagination and at least one filter work on `GET /tasks`, with a bounded `limit`.
- [ ] `/docs` (OpenAPI/Swagger) reviewed by hand; at least two concrete documentation gaps identified and fixed.
- [ ] New pytest tests cover CRUD, auth, ownership, and pagination/filter edge cases, and pass in CI-equivalent local run (`ruff`, `mypy`, `pytest` all green via pre-commit/tox/nox as already configured).
