# Chapter 11 Project — Harden TaskFlow

## Where TaskFlow stands entering Chapter 11

TaskFlow is deployed (Chapter 9), observable via structured logs, correlation IDs, health checks, and metrics (Chapter 10). It has JWT auth and a layered architecture, but it has not yet been through a dedicated security pass — dependencies haven't been audited for known CVEs, there is no regression test proving SQL-injection safety, and there's no automated process keeping dependencies patched over time.

## What to add this chapter

1. **Run `pip-audit` and fix flagged vulnerabilities.**
   - Run `pip-audit` against TaskFlow's full dependency set (and any earlier standalone projects from previous chapters, per the syllabus's "Do This").
   - For each flagged package, upgrade to the recommended fixed version, re-run the test suite to confirm nothing broke, and re-run `pip-audit` to confirm the flag is gone.
   - Add a `pip-audit` step to the GitHub Actions pipeline from Chapter 9 (a new job, or folded into `lint-and-type-check`) so future vulnerable dependencies are caught automatically on every push, not just this one time.

2. **Deliberately introduce and then fix a SQL-injection-vulnerable endpoint, with a regression test.**
   - On a throwaway branch, add (or modify) a search/filter endpoint (e.g., `GET /tasks/search?q=...`) that builds its query via raw string interpolation instead of the existing repository layer's parameterized/ORM query pattern.
   - Demonstrate the exploit locally (e.g., a crafted `q` value that returns unintended rows or errors the query in a revealing way) and record it (a short note, not committed as a permanent vulnerability).
   - Fix it by routing the search back through SQLAlchemy's query-builder API (never raw string-interpolated SQL), consistent with the rest of the repository layer.
   - Add a `pytest` regression test that sends the exact malicious payload used in the exploit and asserts the endpoint behaves safely (no data leakage, no query error) — this test must stay in the suite permanently as a guard against reintroducing the bug.

3. **Verify and harden authentication/authorization.**
   - Confirm passwords are hashed with `argon2` or `bcrypt` (not a fast general-purpose hash), and confirm no plaintext password or the JWT signing secret is ever logged (cross-check against Chapter 10's structured logging — a request/response logging middleware is a common place secrets leak accidentally).
   - Audit at least one "fetch by ID" endpoint (e.g., `GET /tasks/{task_id}`) for broken access control — confirm it verifies the requesting user owns the resource, not just that they're authenticated — and add a test proving a different authenticated user is rejected.

4. **Lock down CORS.** Replace any wildcard `allow_origins=["*"]` with an explicit, environment-driven allow-list of real frontend origins (dev vs prod), since TaskFlow is now publicly deployed and reachable from arbitrary websites' JavaScript otherwise.

5. **Set up Dependabot (or Renovate) on the TaskFlow GitHub repository.**
   - Add `.github/dependabot.yml` configured for the `pip` ecosystem with a weekly schedule.
   - Confirm it opens at least one pull request (even a routine minor-version bump) and that the existing CI pipeline from Chapter 9 runs automatically against it, so a merge only happens once lint/type-check/tests pass.

## Definition of done

- [ ] `pip-audit` reports zero known vulnerabilities against TaskFlow's current dependency set, and runs automatically in CI on every push.
- [ ] A SQL-injection vulnerability was deliberately introduced, demonstrated, fixed via parameterized/ORM queries, and is now permanently guarded by a regression test using the original exploit payload.
- [ ] All passwords are hashed with argon2/bcrypt; no password, JWT secret, or other credential appears in logs.
- [ ] At least one resource-fetching endpoint has an explicit, tested ownership/authorization check beyond plain authentication.
- [ ] CORS is restricted to an explicit allow-list of real origins, not `*`, in the deployed configuration.
- [ ] Dependabot (or Renovate) is active on the repository and has opened at least one dependency-update pull request that runs through the existing CI pipeline.
- [ ] You can explain, in your own words, the difference between CORS and CSRF and why TaskFlow's JWT-header auth reduces (but doesn't eliminate the need to think about) CSRF exposure.
