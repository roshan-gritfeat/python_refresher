# Chapter 8 Project — TaskFlow: Design Patterns & Architecture

## Where TaskFlow stands entering this chapter

By the end of Chapter 7, TaskFlow is a FastAPI application with JWT-protected REST endpoints, a SQLAlchemy/SQLite persistence layer with Alembic migrations, a Redis cache, an async enrichment feature, a multiprocessing-based bulk analytics job, and a fixed, tested race condition — all under pytest, ruff, mypy, and pre-commit. However, up to this point business logic, database access, and API concerns have likely been intermixed directly inside FastAPI route handlers and SQLAlchemy models, which is normal for a project built incrementally but is now a liability: business rules are hard to test without spinning up a DB, and framework code is tangled with domain rules.

## What to add this chapter

Refactor TaskFlow into clean/hexagonal layers — domain, service, repository, API — so business logic no longer imports FastAPI or SQLAlchemy, and prove the decoupling by running the same behavior against a real SQLite repository and a fake in-memory repository.

### 1. Introduce a domain layer

- Create a `taskflow/domain/` module containing plain Python objects (dataclasses, not SQLAlchemy models) representing core concepts: `Task` (entity, has an identity/ID), `Priority` or `TaskStatus` (value objects, likely enums), and any business rules as functions/methods that operate purely on these objects — e.g., "a task cannot transition from `done` back to `todo`," "overdue tasks are those past their due date and not `done`."
- Nothing in this module should import `fastapi`, `sqlalchemy`, `pydantic`, or `redis`. Enforce this with a simple test (e.g., inspecting imports) or a `ruff`/architecture-lint rule if you want to be strict about it.

### 2. Define repository interfaces (ports) and adapters

- Define a `TaskRepository` Protocol/ABC in the domain (or an `interfaces` module) with methods like `add`, `get`, `list_for_user`, `update`, `delete`.
- Implement `SqlAlchemyTaskRepository` (adapter) wrapping the existing Chapter 5 SQLAlchemy models/session, translating between ORM rows and domain `Task` objects at the boundary.
- Implement `InMemoryTaskRepository` (adapter) backed by a plain dict, implementing the exact same interface, for use in tests.

### 3. Introduce a service layer

- Create `taskflow/service/` (or `application/`) containing use-case functions/classes (e.g., `TaskService.create_task(...)`, `TaskService.complete_task(...)`, `TaskService.list_overdue(user_id)`) that orchestrate domain rules and call the repository interface — never a concrete repository class, never raw SQL, never FastAPI request/response objects.
- Move business logic currently living in route handlers (validation beyond simple type-checking, status-transition rules, ownership checks) into this layer.

### 4. Slim down the API layer

- FastAPI route handlers should now only: parse/validate the HTTP request (Pydantic models), call the service layer, and translate the result (or a domain exception) into an HTTP response. No business logic or direct DB/session access should remain in route handlers.
- Wire dependency injection so the concrete repository (`SqlAlchemyTaskRepository`) is provided to the service layer via FastAPI's `Depends`, keeping the choice of repository implementation external to the service/domain code.

### 5. Prove the decoupling with tests

- Write a shared test suite (parametrized or a base test class) that exercises `TaskService` behavior (create, complete, list overdue, ownership enforcement) and runs it twice: once wired to `InMemoryTaskRepository`, once wired to `SqlAlchemyTaskRepository` (against a real, ephemeral SQLite DB). Both runs should pass identically.
- Add a unit test on the domain layer alone (no repository, no DB, no mocks) proving business rules work in complete isolation, e.g., "an overdue check returns true for a past-due, non-done task and false otherwise," constructed purely from Python objects.

### 6. Clean up and re-verify

- Update the async enrichment and multiprocessing analytics features from Chapter 7 to call into the service layer rather than reaching into the DB or domain logic directly, if they currently bypass it.
- Re-run the full pytest suite, ruff, and mypy to confirm the refactor didn't change external behavior — this is a refactor, not a rewrite; the API's observable behavior (status codes, response shapes) from Chapter 6 should be unchanged.

## Definition of done

- [ ] A `domain` module exists with plain Python entities/value objects and business rules, with zero imports of FastAPI, SQLAlchemy, Pydantic, or Redis.
- [ ] A `TaskRepository` interface (Protocol/ABC) is defined, with at least two implementations: SQLAlchemy-backed and in-memory.
- [ ] A `service` layer exists containing use-case logic, depending only on the repository interface, not on any concrete repository or the web framework.
- [ ] FastAPI route handlers are reduced to request/response translation plus calls into the service layer — no business logic or raw DB access remains in routes.
- [ ] The same service-layer test suite passes unchanged against both the in-memory and SQLAlchemy repository implementations.
- [ ] At least one domain-only unit test exists that requires no DB, no mocks, and no framework imports to run.
- [ ] Full pytest suite, ruff, and mypy pass; the API's external behavior (status codes, response shapes from Chapter 6) is unchanged after the refactor.
