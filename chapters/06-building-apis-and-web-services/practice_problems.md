# Chapter 6 — Practice Problems

Standalone exercises, independent of TaskFlow. Ordered easy to hard.

1. **Hello, resource** — Build a FastAPI app with a single `GET /ping` endpoint returning `{"status": "ok"}`.
   *Focus: minimal FastAPI app structure, `uvicorn` run command.*

2. **Typed request body** — Build a `POST /books` endpoint that accepts a Pydantic model with `title: str`, `author: str`, `year: int` and echoes it back with a generated `id`.
   *Hint: use `Field` constraints (e.g., `year` between 1450 and current year) and observe the automatic 422 response for bad input.*

3. **Correct status codes** — Extend the `/books` API with `GET /books/{id}` that returns `404` for a missing book and `200` for a found one, using a proper `HTTPException`.
   *Focus: don't return `200` with an error message in the body.*

4. **Separate DTOs** — Given a single `Book` model that has both `title` and an internal `internal_notes` field, create distinct `BookCreate` and `BookRead` Pydantic models so `internal_notes` never appears in API responses.
   *Hint: this is the read/write model split — think about why reusing one model for both is a leak.*

5. **Pagination** — Implement `GET /books?offset=0&limit=10` over an in-memory list of 100+ books, with sane defaults and an upper bound on `limit`.
   *Focus: reject or clamp a `limit=100000` request rather than silently serving it.*

6. **Filtering + validation** — Add `GET /books?author=<name>&year_from=<int>&year_to=<int>` filtering, returning `400` for a filter combination that makes no sense (e.g., `year_from > year_to`).
   *Hint: validate cross-field constraints, not just individual field types.*

7. **JWT login flow** — Build `POST /login` that checks a hardcoded username/password and returns a signed JWT, and a `GET /me` endpoint that requires a valid `Authorization: Bearer <token>` header to respond.
   *Focus: use `python-jose` or `pyjwt`; set and check token expiry.*

8. **Authorization vs authentication** — Extend problem 7 so `GET /admin/stats` requires not just a valid JWT but a `role: admin` claim in the token payload, returning `403` (not `401`) for a valid-but-unprivileged user.
   *Hint: distinguish "who are you" (401) from "what can you do" (403).*

9. **Consistent error contract** — Add a global exception handler so every error response (validation errors, `HTTPException`s, and unhandled exceptions) is reshaped into one consistent JSON schema, e.g. `{"code": ..., "message": ..., "details": ...}`.
   *Focus: never leak a raw Python traceback to the client in the response body.*

10. **Idempotency key** — Implement `POST /orders` so that sending the same `Idempotency-Key` header twice returns the *original* created order instead of creating a duplicate.
    *Hint: you'll need somewhere to remember keys you've already seen — even an in-memory dict is fine for the exercise.*

11. **Rate limiting** — Add a simple in-memory (or Redis-backed) rate limiter middleware that returns `429` with a `Retry-After` header once a client exceeds N requests per minute.
    *Focus: think about what identifies "a client" (IP? API key?) and what happens at the reset boundary.*

12. **API versioning migration** — Given a `/v1/tasks` endpoint that returns `due_date` as a string, add a `/v2/tasks` endpoint that returns it as a structured `{"date": ..., "timezone": ...}` object, while keeping `/v1` working unchanged for existing clients.
    *Hint: consider whether to duplicate route logic or share an internal service function between versions.*
