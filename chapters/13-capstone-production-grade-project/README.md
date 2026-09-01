# Chapter 13 — Capstone: The Production-Grade Project

## Goal

Prove everything from Chapters 0–12 works together, not just in isolation. This is not a new set of concepts to learn — it's an integration exercise. The bar is: **could you open-source this project, link it in a job application, and defend every architectural decision in a technical interview?**

Build (or finish building, via TaskFlow) one real system that satisfies all ten requirements below. Each one is restated here with what "production-grade" actually means for it — not just "has it," but "is it good enough that a skeptical senior engineer would sign off on it."

---

## The 10 Capstone Requirements

### 1. A `src`-layout installable package, properly versioned

It's not enough to have a `src/` folder. "Properly versioned" means: a single source of truth for the version number (in `pyproject.toml`, ideally not hand-edited in multiple places), semantic versioning discipline (breaking changes bump major, features bump minor, fixes bump patch), and `pip install .` working from a completely clean environment with no manual fixes. A stranger should be able to clone the repo and install it without knowing anything about your machine's setup.

### 2. A REST (or GraphQL) API with authentication

Production-grade auth means more than "there's a login endpoint." Passwords are hashed (bcrypt/argon2, never stored plain or reversibly encrypted), tokens (JWTs or sessions) have real expiry and a way to invalidate them, protected endpoints actually reject unauthenticated and unauthorized requests (tested, not assumed), and the API surface is consistent — predictable status codes, consistent error response shape, sensible resource naming.

### 3. A relational database with migrations

Every schema change since the project's first commit should be represented as a migration (Alembic or equivalent), not as manual `ALTER TABLE` statements someone ran once. Migrations should be reversible where feasible, and a fresh database should reach the current schema by running migrations from zero — no "also run this SQL script by hand" step hiding in a README.

### 4. A test suite (unit + integration) with CI running it on every push

Unit tests exercise business logic in isolation (fast, no real DB/network). Integration tests exercise real boundaries (a real test database, real HTTP calls against a test client). Both run automatically in CI on every push and every PR — not "I run pytest locally before I remember to." A red CI check should actually block a merge, not just be decorative.

### 5. Linting, formatting, and type-checking enforced via pre-commit + CI

"Enforced" is the key word — not "the tools are installed" but "a commit that violates them cannot land." Pre-commit hooks catch issues before they're even committed locally; CI catches anything that slipped through (e.g., someone skipped hooks with `--no-verify`). Type-checking (mypy/pyright) should run at a strictness level that actually catches real bugs, not one so loose it's decorative.

### 6. Structured logging and a health-check/metrics endpoint

Logs are structured (JSON or equivalent), include correlation/request IDs so a single request's full story can be traced through logs, and use log levels meaningfully (not everything at INFO). A health-check endpoint reports real dependency status (DB reachable, not just "process is up"). A metrics endpoint exposes at minimum request counts, latencies, and error rates in a scrapeable format (Prometheus-style).

### 7. Dockerized, with docker-compose for local dev

The Dockerfile is multi-stage (build dependencies don't bloat the final runtime image), uses a `.dockerignore`, and runs as a non-root user. `docker-compose up` brings up the full local stack (app + database + any cache/broker) with zero manual steps beyond maybe copying a `.env.example` to `.env`.

### 8. Deployed to a real (even free-tier) cloud environment

The app is actually reachable over the internet, not just "deployable in theory." Deployment is automated (part of the CI/CD pipeline, not a manual `scp` or console click), and environment configuration (secrets, DB URL, etc.) is injected via environment variables or a secrets manager — never hardcoded or committed.

### 9. A clean architecture separating domain logic from framework/infrastructure

Business logic (domain entities, core rules) does not import FastAPI, SQLAlchemy, or any infrastructure concern directly. You should be able to swap the web framework or the database library without touching the domain layer — and ideally you've actually proven this at some point (e.g., swapping SQLite for an in-memory fake repository in tests, as suggested back in Chapter 8).

### 10. A README that lets a stranger set it up in under 10 minutes

This means: prerequisites are listed explicitly (language version, Docker, etc.), setup is a short, copy-pasteable sequence of commands that works verbatim, environment variables are documented with example values, and there's a clear "how to run the tests" and "how to run it locally" section. The real test: hand the README to someone who has never seen the project and time them. If they hit an undocumented step, the README isn't done.

---

## Example Project Ideas

If you'd rather build a fresh capstone instead of finishing TaskFlow, the syllabus suggests:

- **A URL shortener with analytics** — short-link generation, redirect handling, click tracking, and an analytics dashboard/API (clicks over time, referrers, geography).
- **A personal finance tracker with bank-import parsing** — ingesting bank statement exports (CSV/OFX), categorizing transactions, and reporting on spending over time.
- **A job-board scraper + API + alerting system** — scraping job postings, deduplicating and normalizing them, exposing a search API, and alerting users when new matches appear.
- **A multi-tenant note-taking SaaS** — notes/notebooks scoped per tenant, sharing and permissions, and the multi-tenancy data-isolation concerns that come with it.

Any of these can satisfy all 10 requirements just as well as TaskFlow — pick whichever keeps you motivated to actually finish it and defend it in an interview.

---

## You'll Know You're Ready When...

You could open-source this project, link it in a job application, and defend every architectural decision in a technical interview — including the ones you'd do differently next time.
