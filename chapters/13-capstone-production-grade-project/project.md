# Chapter 13 — Project: TaskFlow Final Integration Checklist

## Do This

Go through the 10 capstone requirements one by one against the TaskFlow project you've accumulated across Chapters 0–12. For each, perform the concrete verification step listed — don't just eyeball the code and assume it's fine. Fix anything that fails verification before considering the capstone done.

If you'd rather not finish TaskFlow, you may instead pick a fresh capstone from the syllabus's example project ideas — a URL shortener with analytics, a personal finance tracker with bank-import parsing, a job-board scraper + API + alerting system, or a multi-tenant note-taking SaaS — and run this same checklist against that project instead once it's built.

---

## Final Integration Checklist

### 1. `src`-layout installable package, properly versioned

- [ ] Confirm `pyproject.toml` has a single `version` field and nothing else in the repo hardcodes a conflicting version.
- [ ] **Verify:** delete your virtual environment, create a brand new one, run `pip install .` from a clean clone, and confirm it succeeds with zero manual intervention.

### 2. REST API with authentication

- [ ] Confirm password hashing uses bcrypt or argon2, and tokens have expiry.
- [ ] **Verify:** write (or run existing) tests that hit a protected endpoint with no token, an expired token, and a valid token — confirm you get 401/403, 401, and 200 respectively.

### 3. Relational database with migrations

- [ ] Confirm every schema change in the project's history has a corresponding Alembic migration.
- [ ] **Verify:** drop the local database entirely, run `alembic upgrade head` against a fresh empty database, and confirm the resulting schema matches what the app expects (app boots and basic CRUD works).

### 4. Test suite (unit + integration) with CI running it on every push

- [ ] Confirm both unit and integration tests exist and are tagged/separated clearly.
- [ ] **Verify:** open your CI provider's run history and confirm the test job ran and passed on the most recent push to a feature branch and on the most recent merge to main — not just "it works when I run it locally."

### 5. Linting, formatting, and type-checking enforced via pre-commit + CI

- [ ] Confirm `.pre-commit-config.yaml` includes ruff (or equivalent) and mypy/pyright.
- [ ] **Verify:** intentionally introduce a formatting violation and a type error in a throwaway branch, attempt to commit — confirm pre-commit blocks it locally — then bypass hooks with `--no-verify` and push, and confirm CI independently catches and fails the same issues.

### 6. Structured logging and a health-check/metrics endpoint

- [ ] Confirm logs are JSON-structured with a request/correlation ID field.
- [ ] **Verify:** make a request to the running app, grep the logs for that request's correlation ID, and confirm you can see its full lifecycle. Then hit `/health` and `/metrics` and confirm health reflects real DB connectivity (stop the DB and confirm health goes unhealthy) and metrics show real request counts.

### 7. Dockerized, with docker-compose for local dev

- [ ] Confirm the Dockerfile is multi-stage and runs as a non-root user.
- [ ] **Verify:** run `docker compose down -v` to fully tear down local state, then `docker compose up` from a clean clone (with only `.env` populated from `.env.example`) and confirm the full stack (app + db + any cache/broker) comes up and the app is reachable, with zero additional manual steps.

### 8. Deployed to a real (even free-tier) cloud environment

- [ ] Confirm the deployed URL is currently live.
- [ ] **Verify:** from a machine/network you don't normally use (or just a fresh browser session with no local dev server running), hit the deployed URL's health endpoint and one real API endpoint, and confirm both respond correctly.

### 9. Clean architecture separating domain logic from framework/infrastructure

- [ ] Confirm domain/service-layer modules have no imports of FastAPI, SQLAlchemy, or other infrastructure packages.
- [ ] **Verify:** run a grep (e.g., `grep -r "import fastapi\|import sqlalchemy" src/taskflow/domain/`) and confirm zero hits. Bonus: confirm your test suite actually swaps in an in-memory fake repository somewhere without touching domain logic, as set up back in Chapter 8.

### 10. README that lets a stranger set it up in under 10 minutes

- [ ] Confirm prerequisites, setup commands, environment variables, and test/run instructions are all present.
- [ ] **Verify:** hand the README (and only the README — no verbal help) to someone who has never seen the project, or simulate this yourself on a completely clean machine/VM, and time how long it takes to get the app running. If it takes over 10 minutes or requires an undocumented step, fix the README.

---

## Done means done

Once every box above is checked with its verification step actually performed (not assumed), TaskFlow — or your chosen alternative capstone — is ready to be open-sourced, linked in a job application, and defended in a technical interview.
