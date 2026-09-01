# Chapter 5 — Databases & Persistence

Goal: real applications need to remember things.

---

## SQL Fundamentals Refresher: Joins, Indexes, Transactions, Normalization Basics

**Joins** combine rows from two or more tables based on a related column. An `INNER JOIN` returns only rows with matches in both tables; a `LEFT JOIN` returns all rows from the left table plus matches from the right (with `NULL`s where there's no match) — the most common source of confusion is expecting a `LEFT JOIN` to filter like an `INNER JOIN` when a `WHERE` clause on the right table's column accidentally turns it into one (a `WHERE right.col = X` excludes the `NULL` rows a `LEFT JOIN` was meant to preserve; put such conditions in the `ON` clause instead).

```sql
SELECT tasks.title, projects.name
FROM tasks
LEFT JOIN projects ON tasks.project_id = projects.id;
```

**Indexes** are auxiliary data structures (typically B-trees) that let the database find rows matching a condition without scanning the whole table. They dramatically speed up `WHERE`, `JOIN`, and `ORDER BY` on the indexed column(s), at the cost of extra storage and slower writes (every `INSERT`/`UPDATE` must also update the index). The common pitfall is either under-indexing (slow queries on large tables doing full scans) or over-indexing (indexing every column "just in case," which slows down writes for indexes that are never used in a `WHERE`/`JOIN`/`ORDER BY`). Always index foreign key columns you join on and columns you filter on frequently; verify with `EXPLAIN QUERY PLAN` (SQLite) or `EXPLAIN ANALYZE` (Postgres) that a query is actually using the index you expect.

**Transactions** group multiple statements into one atomic unit: either all of them commit, or (on error/rollback) none of them do. This is essential whenever multiple writes must stay consistent with each other — e.g., moving a task between two lists (delete from one, insert into the other) must not leave the database in a half-done state if the process crashes between the two statements. The classic ACID properties describe what a transaction guarantees: **Atomicity** (all-or-nothing), **Consistency** (valid state to valid state), **Isolation** (concurrent transactions don't see each other's half-finished work), **Durability** (once committed, survives a crash).

**Normalization** is the discipline of structuring tables to avoid redundant data and the update anomalies that come with it. First normal form (1NF) requires atomic column values (no comma-separated lists in a single column); second and third normal form (2NF/3NF) progressively remove columns that depend on only part of a key, or on a non-key column, moving them into their own table instead. In practice: if you find yourself updating the same fact (a project's name, a user's email) in multiple rows/tables whenever it changes, that data probably belongs in its own table referenced by foreign key. Real-world schemas sometimes deliberately **denormalize** (duplicate data) for read performance — a valid tradeoff, but one that should be a conscious decision, not an accident.

---

## Working with sqlite3 Directly (to Understand the Primitives)

Python's standard library `sqlite3` module gives direct access to a real SQL database with zero external dependencies, which makes it the ideal tool for *learning what an ORM does underneath* before reaching for one.

```python
import sqlite3

conn = sqlite3.connect("tasks.db")
conn.execute(
    "CREATE TABLE IF NOT EXISTS tasks (id INTEGER PRIMARY KEY, title TEXT NOT NULL, done INTEGER DEFAULT 0)"
)
conn.execute("INSERT INTO tasks (title) VALUES (?)", ("Buy milk",))
conn.commit()

for row in conn.execute("SELECT id, title, done FROM tasks WHERE done = ?", (0,)):
    print(row)

conn.close()
```

Key primitives to internalize: a **connection** represents a link to the database file/server; a **cursor** (or, for simple cases, the connection's own `execute`) sends statements and retrieves results; **parameterized queries** (the `?` placeholders above) are non-negotiable — never use an f-string to build SQL from user input, since that is exactly how SQL injection vulnerabilities happen (covered in depth in Chapter 11). `sqlite3` does not auto-commit by default for DML, so a common bug is forgetting `conn.commit()` and wondering why the data "disappeared" — using the connection as a context manager (`with conn:`) auto-commits/rolls-back the block. Understanding these primitives is what lets you later read an ORM's generated SQL and actually understand what it's doing, and diagnose it when it does something unexpected.

---

## ORMs: SQLAlchemy Core vs ORM Layer, Django ORM (If Going Django Route)

An **Object-Relational Mapper (ORM)** lets you work with database rows as Python objects instead of hand-writing SQL strings, and manages connection handling, query building, and (often) migrations for you.

**SQLAlchemy** actually ships two layers: **Core** is a SQL expression language — you build queries with Python objects (`select()`, `insert()`, table/column objects) that map closely to the SQL they generate, without necessarily mapping rows to full Python classes. The **ORM** layer sits on top of Core and maps database tables to Python classes (models), letting you write `session.query(Task).filter_by(done=False)` or, in modern (2.0-style) SQLAlchemy, `session.execute(select(Task).where(Task.done == False))`, and gives you relationship traversal (`task.project.name`) instead of manual joins.

```python
from sqlalchemy import String, ForeignKey
from sqlalchemy.orm import Mapped, mapped_column, DeclarativeBase, relationship

class Base(DeclarativeBase):
    pass

class Project(Base):
    __tablename__ = "projects"
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(100))

class Task(Base):
    __tablename__ = "tasks"
    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(200))
    done: Mapped[bool] = mapped_column(default=False)
    project_id: Mapped[int] = mapped_column(ForeignKey("projects.id"))
    project: Mapped["Project"] = relationship()
```

**Django's ORM** is tightly integrated into the Django framework (models, migrations, and admin all built around it) and is simpler to get started with if you're already using Django, but is less useful standalone outside a Django project. SQLAlchemy is framework-agnostic and works equally well in a CLI tool, a FastAPI service, or a script.

**Common pitfalls with any ORM:** treating it as a black box that means you never need to understand SQL (you will eventually hit a slow query only diagnosable by reading the generated SQL); "lazy loading" relationships accidentally triggering the N+1 problem (see below); and forgetting that an ORM session/unit-of-work has its own lifecycle (when do writes actually flush to the DB, when does a transaction commit) that differs from a raw `sqlite3` connection.

---

## Migrations: alembic

A **migration** is a versioned, incremental change to a database schema (add a column, create a table, add an index) that can be applied and — ideally — reversed, tracked in source control alongside the code that depends on it. Without migrations, keeping a team's local databases, staging, and production in sync as the schema evolves becomes error-prone guesswork ("did you remember to add that column on your machine?").

**Alembic** is SQLAlchemy's migration tool. It can auto-generate a migration script by diffing your ORM models against the current database schema (`alembic revision --autogenerate -m "add priority column"`), producing an `upgrade()`/`downgrade()` pair of functions you review and commit. Running `alembic upgrade head` applies all pending migrations in order; `alembic downgrade -1` reverts the most recent one.

```python
def upgrade():
    op.add_column("tasks", sa.Column("priority", sa.Integer(), server_default="1"))

def downgrade():
    op.drop_column("tasks", "priority")
```

**Common pitfalls:** trusting autogenerate blindly — it can miss things (renamed columns look like a drop+add, some constraint changes aren't detected) and always needs a human review pass before committing; editing an already-applied migration instead of writing a new one (this desyncs anyone who already ran it); and forgetting that a migration touching a large production table can lock it for a long time — for big tables, migrations sometimes need to be split into backward-compatible steps (add nullable column → backfill → make non-nullable, deployed separately) rather than one atomic change.

---

## Connection Pooling, N+1 Query Problem, Query Optimization Basics

**Connection pooling** reuses a small set of already-open database connections across requests/operations instead of opening and closing a new connection every time (opening a connection has real latency and resource cost, especially over a network to a remote DB server). SQLAlchemy manages a connection pool automatically under its `Engine`; the main tuning knobs are pool size and timeout, relevant once an application has meaningful concurrent load.

**The N+1 query problem** is one of the most common ORM-related performance bugs: fetching a list of N parent objects, then accessing a related object/collection on each one triggers N *additional* queries (one per parent) instead of one efficient batched query — 1 query to get the tasks, plus N queries (one per task) to lazily load each task's project.

```python
# N+1: one query for tasks, then one more query PER task to fetch its project
tasks = session.execute(select(Task)).scalars().all()
for t in tasks:
    print(t.project.name)  # triggers a lazy-load query each iteration
```

The fix is **eager loading** — telling the ORM up front to fetch the related rows in the same query (a JOIN) or in one batched follow-up query:

```python
from sqlalchemy.orm import joinedload

tasks = session.execute(select(Task).options(joinedload(Task.project))).scalars().all()
for t in tasks:
    print(t.project.name)  # no extra query — already loaded
```

**Query optimization basics:** select only the columns you need rather than whole objects when you just need a couple of fields; use `EXPLAIN`/`EXPLAIN ANALYZE` to see whether an index is actually being used; batch bulk inserts/updates instead of looping row-by-row; and be skeptical of "just add an index" as a universal fix — indexes have write-cost tradeoffs and don't help every query shape (e.g., a `LIKE '%text%'` leading wildcard can't use a standard B-tree index).

---

## NoSQL Basics: When Mongo/Redis Make Sense vs Relational DBs

Relational databases (SQLite, Postgres, MySQL) enforce a schema up front and excel at relationships, joins, and transactional consistency — the right default for most application data with clear structure and relationships (users, orders, tasks-with-projects).

**Document stores** (MongoDB) store flexible, schema-less (or loosely-schema'd) JSON-like documents, which suit data whose shape varies a lot between records or evolves quickly, and where you mostly fetch whole documents rather than doing complex cross-entity joins. The tradeoff is weaker native support for relational integrity/joins across documents.

**Key-value / in-memory stores** (Redis) trade durability and query flexibility for speed, and are used less as a system of record and more for: caching (see below), session storage, rate-limiting counters, pub/sub messaging, and leaderboards/queues where the access pattern is simple key lookups at very high throughput.

The decision isn't "NoSQL is faster/more modern" — it's about access patterns: relational for structured, relationship-heavy, transactional data; document stores for flexible/variable-shape data accessed mostly as whole records; key-value/in-memory stores for ephemeral, high-throughput, simple-lookup data. Many real systems use more than one (a relational DB as the system of record, Redis as a cache/session store in front of it) — this is normal, not a sign of indecision.

---

## Caching Strategies: In-Memory, Redis, Cache Invalidation Basics

Caching stores the result of an expensive operation (a slow query, a computed aggregate) so a subsequent identical request can be served near-instantly instead of redoing the work.

**In-memory caching** (a plain Python dict, or `functools.lru_cache` for pure functions) is the simplest option, but it's local to a single process — useless across multiple app instances/workers, and lost on restart.

**Redis** as a cache is shared across all instances of your application and persists (optionally) across restarts, at the cost of a network hop (still far faster than re-running an expensive DB query) and needing to serialize/deserialize values (commonly JSON) going in and out.

**Cache invalidation** is the hard part — famously one of the "two hard things in computer science." Common strategies: **TTL (time-to-live)** expiration (simplest — accept slightly stale data for a bounded window, e.g., "cache for 60 seconds"); **explicit invalidation** (delete/update the cache key whenever the underlying data changes — more accurate but requires remembering to do it at every write path that touches that data, which is easy to miss); and **cache-aside** (the most common pattern: on read, check cache first, on miss query the DB and populate the cache; on write, update the DB and either delete or update the corresponding cache key).

```python
def get_task_count(project_id: int) -> int:
    key = f"project:{project_id}:task_count"
    cached = redis_client.get(key)
    if cached is not None:
        return int(cached)
    count = db_count_tasks(project_id)
    redis_client.setex(key, 60, count)  # cache for 60 seconds
    return count
```

**Common pitfalls:** caching data that changes so often the cache is almost always stale/invalidated anyway (little benefit, added complexity); forgetting to invalidate on every write path that affects the cached value, leading to subtly stale reads that are hard to reproduce; and caching at too coarse a granularity (one giant cached blob invalidated by any unrelated change) versus too fine (so many cache keys that hit rates drop and invalidation logic gets complex) — the right granularity matches your actual read/write access pattern.
