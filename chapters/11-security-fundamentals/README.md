# Chapter 11 — Security Fundamentals

**Goal:** don't ship the vulnerabilities that make security engineers cry. This chapter is about instinct-building — the goal isn't to become a security specialist, it's to never again accidentally write the mistakes on this list.

---

## OWASP Top 10, applied to Python web apps

The OWASP Top 10 is a periodically-updated, industry-consensus list of the most common and impactful web application vulnerabilities. It matters because it's *empirical* — it's not theoretical risk, it's what actually keeps compromising real production systems. A few entries especially relevant to a FastAPI service like TaskFlow:

- **Injection** (SQL injection, command injection) — untrusted input is concatenated into a command interpreter without sanitization. Covered in depth below.
- **Broken authentication** — weak password policies, tokens that never expire, session fixation, or (very commonly) rolling your own crypto/hashing instead of using a vetted library.
- **Broken access control** — an endpoint correctly checks that a user is *authenticated* but forgets to check they're *authorized* for the specific resource (e.g., User A can fetch User B's tasks just by guessing the task ID in the URL). This is one of the most common real-world bugs and is easy to miss because the endpoint "works" for the happy path.
- **Security misconfiguration** — default credentials left in place, verbose error pages leaking stack traces to end users, permissive CORS in production, debug mode left on.
- **Vulnerable and outdated components** — depending on libraries with known CVEs and never checking. Covered under dependency scanning below.

The pitfall with OWASP-as-a-checklist is treating it as something you audit once before "launch" — in reality, every new endpoint you write is a fresh opportunity to reintroduce any of these, so the goal is to internalize the *patterns* (validate input, check both authentication and authorization, never trust the client) rather than memorizing a list.

---

## Input validation and sanitization

**Never trust the client.** Any data arriving from outside your process boundary — request bodies, query parameters, headers, even data from a "trusted" internal service — must be validated before use. In FastAPI this is largely handled structurally by **Pydantic models**: declaring a request body as a Pydantic model with typed fields and constraints (`str = Field(max_length=200)`, `EmailStr`, custom validators) means malformed or unexpected input is rejected with a `422` before it ever reaches your business logic.

Validation and sanitization are related but distinct: *validation* rejects input that doesn't conform to expectations (e.g., an age of `-5`); *sanitization* transforms input to make it safe without necessarily rejecting it (e.g., stripping HTML tags from a free-text field before storing it, to prevent stored XSS if that field is ever rendered in a browser).

Common pitfall: validating at the API boundary but then trusting that same data unconditionally deeper in the system — e.g., a service-layer function that's also callable from a background job or CLI script without going through the Pydantic model at all. Validation should happen at every trust boundary the data crosses, not just the outermost one.

---

## SQL injection prevention

SQL injection happens when untrusted input is concatenated directly into a SQL query string, allowing an attacker to change the query's *structure*, not just its data. The classic example:

```python
# VULNERABLE — never do this
query = f"SELECT * FROM users WHERE username = '{username}'"
cursor.execute(query)
```

If `username` is `' OR '1'='1`, the resulting query becomes `SELECT * FROM users WHERE username = '' OR '1'='1'`, returning every row — an authentication bypass. Worse payloads can chain a `;` and execute arbitrary additional statements, including `DROP TABLE`.

The fix is **parameterized queries** — placeholders that the database driver binds separately from the query structure, so user input can never be interpreted as SQL syntax:

```python
# SAFE — parameterized
cursor.execute("SELECT * FROM users WHERE username = ?", (username,))
```

When using an ORM like SQLAlchemy correctly (via its query-building API — `select(User).where(User.username == username)`), parameterization happens automatically and you'd have to go out of your way (e.g., using `text()` with raw string interpolation) to reintroduce the vulnerability. This is "ORM safety": the ORM's abstraction *is* the injection defense, as long as you don't drop down to raw SQL string concatenation for "one quick optimization."

Pitfall: assuming an ORM makes you immune. `session.execute(text(f"... {user_input} ..."))` is exactly as vulnerable as raw `sqlite3` string formatting — the ORM only protects you through its query-builder API, not through raw-SQL escape hatches.

---

## Secrets & credentials handling, password hashing

Passwords must **never** be stored in plaintext or with a fast, general-purpose hash like plain SHA-256/MD5 — those are designed to be *fast*, which is precisely the wrong property for password storage, because it makes brute-forcing stolen hashes cheap. Instead, use a hashing algorithm deliberately designed to be slow and memory-hard: **bcrypt** or **argon2** (the current OWASP-recommended default). These incorporate a per-password salt automatically and a configurable work factor, so verifying a password takes a controlled, non-trivial amount of time — cheap enough for a legitimate login, expensive enough to make offline brute-forcing of a leaked hash database impractical.

```python
from argon2 import PasswordHasher

ph = PasswordHasher()
hashed = ph.hash("user-supplied-password")
ph.verify(hashed, "user-supplied-password")  # raises on mismatch
```

For TaskFlow, this means the JWT auth layer from Chapter 6 should never compare plaintext passwords or store them anywhere — only the hash, generated at signup and verified (not decrypted — hashing is one-way) at login.

Beyond passwords, other credentials (JWT signing secrets, database passwords, third-party API keys) must be injected via environment variables/secret managers (as established in Chapter 9), never hardcoded, and rotated if ever exposed. A leaked JWT signing secret is catastrophic — it lets an attacker forge valid tokens for any user, including admins.

---

## Dependency vulnerability scanning

Your own code isn't the only attack surface — every third-party package you depend on (and every package *those* packages depend on, transitively) is too. **`pip-audit`** scans your installed environment or lock file against known vulnerability databases (primarily the Python Packaging Advisory Database) and reports any installed version with a known CVE, along with the fixed version to upgrade to:

```bash
pip-audit
```

This should run regularly, not just once — new CVEs are discovered in existing packages constantly, so a dependency that was clean last month might not be today. That's what **Dependabot** (built into GitHub) or **Renovate** automate: they periodically scan your `pyproject.toml`/lock file, open a pull request bumping a vulnerable (or simply outdated) dependency to a patched version, and can be configured to auto-merge patch-level bumps that pass CI.

Pitfall: pinning dependencies exactly (`==`) for reproducibility (good) but then never having any automated process to bump those pins when a CVE is announced (bad) — pinning without a renewal mechanism just means you're reproducibly vulnerable.

---

## HTTPS, CORS, CSRF basics for web services

**HTTPS** encrypts traffic between client and server, preventing eavesdropping and tampering in transit — anything sent over plain HTTP (including a JWT in an `Authorization` header) can be trivially intercepted on shared networks. In production this is usually handled at the load balancer/PaaS edge (most platforms provision TLS certificates automatically), but it's worth understanding it's not "the framework's job" — FastAPI itself doesn't terminate TLS in typical deployments.

**CORS (Cross-Origin Resource Sharing)** is a *browser-enforced* mechanism that controls which origins (domains) are allowed to make requests to your API from client-side JavaScript. By default, browsers block cross-origin requests unless your server responds with headers explicitly allowing it (`Access-Control-Allow-Origin`). This is not a security boundary against non-browser clients (`curl` or a script ignores CORS entirely) — it protects browser users from a malicious website silently making authenticated requests to your API using their existing session. Pitfall: setting `Access-Control-Allow-Origin: *` "to make the CORS errors go away" during development and shipping that to production — this permits *any* website to call your API from a logged-in user's browser.

**CSRF (Cross-Site Request Forgery)** is a related but distinct attack: it exploits the browser automatically attaching cookies (including session cookies) to requests, even ones triggered by a malicious third-party page the user happens to have open. If your API relies on cookie-based sessions, a malicious page could trigger a state-changing request (e.g., `POST /tasks/delete`) that the browser dutifully attaches the user's valid session cookie to. The defense is a CSRF token — a value the malicious page can't know — required on state-changing requests. TaskFlow's JWT-in-`Authorization`-header approach (rather than cookies) is inherently far less exposed to CSRF, since a malicious page cannot read or set another origin's `Authorization` header — but this is worth understanding explicitly rather than accidentally: if TaskFlow ever moves to cookie-based auth (e.g., for browser session convenience), CSRF protection becomes mandatory again.

**CORS vs CSRF in one sentence:** CORS controls *whether the browser lets JavaScript read the response* of a cross-origin request; CSRF is about *whether a request gets sent with the user's credentials attached at all*, regardless of whether the attacker can read the response. They are frequently confused because both involve "some other website interacting with your API," but they are different threats with different defenses.
