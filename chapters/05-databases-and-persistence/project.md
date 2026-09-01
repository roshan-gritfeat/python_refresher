# TaskFlow — Chapter 5 Increment: Databases & Persistence

## Where TaskFlow stands

After Chapter 4, TaskFlow has a fully gated development workflow: ruff, mypy, and pre-commit block bad commits, its public API is documented and has generated docs, and it has a task runner (`just check` or equivalent). Its persistence layer, however, is still whatever in-memory or flat-file storage was built in Chapters 0-2 (e.g., an `InMemoryTaskStore` or a JSON file) — there is no real database.

## What to add this chapter

1. **SQLite + SQLAlchemy persistence layer.**
   - Add `sqlalchemy` as a runtime dependency.
   - Define ORM models with modern `Mapped`/`mapped_column` declarative style: a `Task` model (id, title, description, done, priority, created_at, due_date) and a `Project` model (id, name), with a one-to-many relationship (`Project.tasks`, `Task.project`).
   - Implement a `SqlAlchemyTaskStore` (or repository class) that satisfies the same interface your existing storage abstraction already uses (from Chapter 2/3), so the CLI and business logic don't need to change — only the storage implementation swaps in. This is the payoff of having designed a storage abstraction earlier.
   - Wire up a SQLite database file (e.g., `~/.taskflow/taskflow.db` or a configurable path via the settings mechanism from Chapter 2) as the default backing store.

2. **Alembic migrations.**
   - Initialize Alembic (`alembic init migrations`) configured to read the SQLAlchemy models' metadata for autogenerate.
   - Generate and commit an initial migration creating the `projects` and `tasks` tables.
   - Add a second, separate migration that adds a `priority` (or another genuinely new) column to `tasks`, demonstrating the incremental-migration workflow.
   - Document in the README how to run migrations (`alembic upgrade head`) as part of project setup.

3. **Deliberate N+1 query, then fix it.**
   - Implement a CLI command or function (e.g., `taskflow projects --with-counts`) that lists all projects along with each project's task count, initially written in a way that triggers an N+1 query (accessing `project.tasks` in a loop with lazy loading).
   - Demonstrate the problem: with SQLAlchemy engine `echo=True` (or a query-counting test utility), show that listing 10 projects issues 11 queries.
   - Fix it using eager loading (`selectinload`/`joinedload`) or an aggregate query (`GROUP BY` with `func.count`), and demonstrate the query count drops to 1-2.
   - Write a regression test that asserts the query count stays low, so a future change can't silently reintroduce the N+1.

4. **Redis cache layer on a read-heavy operation.**
   - Add `redis` (and `fakeredis` as a test dependency) as dependencies.
   - Identify a read-heavy, relatively stable read in TaskFlow (e.g., "task counts per project" from step 3, or "overdue task count") and wrap it in a cache-aside layer: check Redis first, fall back to the DB query on a miss, populate the cache with a TTL (e.g., 30-60 seconds).
   - Implement invalidation: whenever a task is added/completed/deleted for a project, explicitly delete that project's cached count so stale data isn't served indefinitely.
   - Write tests (using `fakeredis` so no real Redis server is required in CI) proving: (a) a repeated read within the TTL doesn't re-hit the database, and (b) a write invalidates the cache so the next read reflects the change.

5. **Update tests from Chapter 3.**
   - Adjust the existing storage tests to run against the new `SqlAlchemyTaskStore` (using an in-memory SQLite `sqlite:///:memory:` or a temp file per test) instead of (or in addition to) the old in-memory store, proving the storage abstraction really is swappable.

## Definition of done

- [ ] SQLAlchemy models (`Task`, `Project`) defined with type-annotated `Mapped` columns and a working relationship.
- [ ] A `SqlAlchemyTaskStore` implements TaskFlow's existing storage interface and is used as the default backend.
- [ ] Alembic is initialized; at least two migrations exist (initial schema + one incremental column addition) and both apply cleanly via `alembic upgrade head`.
- [ ] A project-task-count feature was built, shown to cause an N+1 query (with evidence — echoed SQL or a query count), then fixed with eager loading or aggregation.
- [ ] A regression test asserts the fixed query stays at a low, fixed query count.
- [ ] A Redis-backed cache-aside layer wraps a read-heavy operation, with a documented TTL.
- [ ] Cache invalidation is wired into every write path that affects the cached value.
- [ ] Tests (using `fakeredis`) prove both the cache-hit-avoids-db-call behavior and the invalidation-on-write behavior.
- [ ] Chapter 3's storage test suite passes against the new SQLAlchemy-backed store.
- [ ] `just check` (lint + typecheck + test, from Chapter 4) still passes with zero errors after all of the above.
