# Chapter 11 — Practice Problems: Security Fundamentals

Standalone exercises, ordered easy to hard.

1. **Identify the vulnerability.** Given a snippet that builds a SQL query with an f-string using unsanitized user input, name the vulnerability class and rewrite it using a parameterized query.
   *Focus: recognizing injection at a glance.*

2. **Hash and verify a password with `argon2` (or `bcrypt`).** Write a script that hashes a password, stores only the hash, and verifies a correct vs incorrect password against it.
   *Hint: use `argon2-cffi`'s `PasswordHasher`, note that `verify` raises rather than returning a boolean on mismatch.*

3. **Break a naive password check.** Given a toy login function that compares passwords with plain SHA-256 and no salt, explain (in writing, no need to actually crack it) why this is insecure even though "it's hashed," and rewrite it using argon2.
   *Focus: fast hashes vs slow/memory-hard hashes.*

4. **Add Pydantic validation to reject bad input.** Given a FastAPI endpoint accepting a raw `dict` body, convert it to a Pydantic model with field constraints (length limits, `EmailStr`, a custom validator) and demonstrate a request that now gets rejected with `422`.
   *Hint: `Field(..., max_length=...)`, `@field_validator`.*

5. **Find a broken access control bug.** Given a toy endpoint `GET /orders/{order_id}` that returns any order by ID with no ownership check, write the fix that verifies the requesting user owns that order, plus a test proving a different user gets `403`/`404`.
   *Focus: authentication (who are you) vs authorization (are you allowed to see this specific resource).*

6. **Run `pip-audit` on a real (or toy) project's dependencies** and report at least one flagged package (or confirm none are flagged), including the recommended fixed version.
   *Hint: run inside the project's virtual environment so it audits the actual installed set.*

7. **Deliberately introduce a SQL-injection-vulnerable endpoint** in a throwaway FastAPI + SQLite app (a search endpoint that string-formats a `LIKE` query), then demonstrate the exploit with a crafted input.
   *Focus: reproduce the vulnerability yourself before fixing it — this cements what "unsafe" actually looks like at the wire level.*

8. **Fix the endpoint from #7** using a parameterized query, and write a regression test (`pytest`) using the exact malicious payload from #7 that asserts the app no longer misbehaves.
   *Hint: assert the response is either empty results or a validation error — not a full table dump.*

9. **Configure CORS correctly for a specific origin.** Given a FastAPI app currently allowing `allow_origins=["*"]`, restrict it to a single named frontend origin and write a test (using `TestClient`) confirming a disallowed origin's preflight is rejected.
   *Focus: understanding this is enforced by browsers, not by your test client directly — read up on what `TestClient` can and can't verify here.*

10. **Explain CORS vs CSRF with a concrete scenario for each.** Write two short scenarios (a few sentences each): one where a misconfigured CORS policy causes a problem, and one where a missing CSRF token causes a problem — for an app that uses cookie-based sessions.
    *Focus: articulating the distinction clearly enough to explain to a teammate.*

11. **Set up Dependabot on a real GitHub repository** (yours or a fork) with a `.github/dependabot.yml` targeting the Python ecosystem, and let it open at least one PR (or simulate by manually bumping a dependency and observing the config would trigger).
    *Hint: `package-ecosystem: "pip"`, `schedule.interval`.*

12. **Audit a full mini-app against the OWASP list.** Given a small (intentionally flawed) FastAPI app provided or written by you with 3–4 planted issues (e.g., no input validation, plaintext password storage, `allow_origins=["*"]`, a raw-SQL endpoint), write up which OWASP Top 10 category each issue falls under and the specific fix for each.
    *Focus: practicing the full triage process end to end, not just fixing one issue in isolation.*
