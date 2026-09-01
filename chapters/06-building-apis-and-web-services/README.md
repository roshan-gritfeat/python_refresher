# Chapter 6 — Building APIs & Web Services

Goal: expose your logic to the world (or your frontend) properly.

## Framework choice: FastAPI vs Django vs Flask

Choosing a web framework is an architectural decision, not a taste preference. The three you'll meet most in Python are:

- **FastAPI** — async-first, built on Starlette + Pydantic, generates OpenAPI docs automatically from your type hints. It is "type-driven": your function signatures *are* your request/response schema. Best when you want speed of iteration, strong typing, and native async support (e.g., calling other services concurrently).
- **Django** — batteries-included. Ships with an ORM, admin panel, auth system, forms, templating, and migrations out of the box. Great for content-heavy or CRUD-heavy apps where you want conventions decided for you. Historically synchronous-first, though Django now has ASGI and async views.
- **Flask** — minimal, unopinionated. You bring your own ORM, validation, and structure. Great for small services or when you need full control, but you own more wiring (and more ways to get it wrong at scale).

**Trade-offs to internalize:**
- FastAPI's type-driven validation reduces a whole class of bugs (mismatched request shapes) but couples your endpoint signatures tightly to Pydantic models — you must be deliberate about not leaking those models into your domain layer (this becomes central in Chapter 8).
- Django's "everything included" approach accelerates CRUD apps but its ORM and app structure are harder to swap out or decouple later; hexagonal architecture is more work to retrofit onto Django.
- Flask gives you freedom, which means your project's quality is bounded by your own discipline, not the framework's.

For TaskFlow, we use **FastAPI** because it plays well with the SQLAlchemy layer, Pydantic gives us free request/response validation, and the automatic OpenAPI docs teach documentation habits early.

**Common pitfall:** picking a framework based on hype rather than the shape of the problem. A content management site full of forms and an admin backend often *is* better served by Django. A small, latency-sensitive JSON API is usually better served by FastAPI.

## REST principles: resources, status codes, idempotency, versioning

REST models your API around **resources** (nouns, not verbs): `/tasks`, `/tasks/{id}`, `/users/{id}/tasks`, not `/getTasks` or `/createTask`. Each resource supports a subset of HTTP verbs (GET, POST, PUT, PATCH, DELETE) whose behavior should match its semantics.

**Status codes** communicate outcome, and using them correctly is part of making an API self-documenting:
- `200 OK` — successful GET/PUT/PATCH.
- `201 Created` — successful POST that created a resource (include a `Location` header pointing to it).
- `204 No Content` — successful action with nothing to return (e.g., DELETE).
- `400 Bad Request` — malformed input.
- `401 Unauthorized` — no/invalid credentials.
- `403 Forbidden` — valid credentials, insufficient permission.
- `404 Not Found` — resource doesn't exist.
- `409 Conflict` — state conflict (e.g., duplicate unique field).
- `422 Unprocessable Entity` — semantically invalid payload (FastAPI uses this automatically for Pydantic validation failures).
- `429 Too Many Requests` — rate limited.
- `500 Internal Server Error` — unhandled server-side failure.

**Idempotency** means calling the same operation multiple times produces the same end state as calling it once. GET, PUT, and DELETE should be idempotent; POST typically is not (each call creates a new resource). This matters enormously for retries: if a client's connection drops after it sends a POST but before it gets a response, blindly retrying can create duplicates. A common pattern is an `Idempotency-Key` header the server uses to deduplicate retried requests.

**Versioning** protects existing clients when you need to make breaking changes. Common strategies:
- URI versioning: `/v1/tasks`, `/v2/tasks` — simple, visible, but can proliferate routes.
- Header versioning: `Accept: application/vnd.taskflow.v2+json` — cleaner URLs, less discoverable.
- Query param versioning: `/tasks?version=2` — easy to add but easy to forget.

**Common pitfall:** modeling actions as resources incorrectly, e.g., `POST /tasks/123/complete` when you could instead `PATCH /tasks/123` with `{"status": "completed"}`. Both are defensible, but pick one convention and apply it consistently across the API.

```python
# Example: idempotent PATCH vs non-idempotent POST
@app.patch("/tasks/{task_id}")
def update_task(task_id: int, payload: TaskUpdate):
    ...  # calling this twice with the same payload leaves the same end state

@app.post("/tasks")
def create_task(payload: TaskCreate):
    ...  # calling this twice creates two tasks unless deduplicated
```

## Request validation with Pydantic models

FastAPI uses **Pydantic** models to validate, parse, and document request/response bodies. You declare the shape you expect, and Pydantic enforces it before your function body ever runs — malformed data never reaches your business logic.

```python
from pydantic import BaseModel, Field

class TaskCreate(BaseModel):
    title: str = Field(min_length=1, max_length=200)
    description: str | None = None
    priority: int = Field(default=0, ge=0, le=5)
```

If a client sends `priority: 10`, Pydantic rejects it with a `422` before your endpoint code runs, and the error message tells the client exactly which field failed and why.

**Why it matters:** validation logic scattered through `if` statements in every endpoint is a maintenance nightmare and easy to get inconsistent. Centralizing it in typed models means the validation rules live in one place, are enforced consistently, and double as documentation.

**Common pitfalls:**
- Reusing the same Pydantic model for input and output. A `TaskCreate` model shouldn't be the same as a `TaskRead` model — the former lacks a server-assigned `id` and `created_at`; exposing internal fields (like a password hash) via a response model reused from a DB model is a classic security leak.
- Over-trusting nested/optional fields: mark fields `Optional` only when genuinely optional, or you'll silently accept missing data that later crashes deeper in the stack.
- Forgetting to separate the **Pydantic model** (API/DTO shape) from the **SQLAlchemy model** (persistence shape). Chapter 8 is explicitly about not letting this coupling metastasize.

## Authentication & authorization: sessions, JWTs, OAuth2 basics

**Authentication** answers "who are you?"; **authorization** answers "what are you allowed to do?" They're often conflated but should be handled as separate concerns.

- **Sessions**: server stores session state (often in Redis or a DB), and the client holds an opaque session ID in a cookie. Simple to reason about and easy to revoke, but requires server-side state and doesn't scale as cleanly across stateless services.
- **JWT (JSON Web Tokens)**: a signed (and optionally encrypted) token containing claims (e.g., `sub`, `exp`, `roles`), which the server can verify without a database lookup. Stateless and scales well horizontally, but revocation is harder (you either keep tokens short-lived, or maintain a blocklist — which reintroduces state).
- **OAuth2**: a delegation protocol — "let app X act on behalf of user Y without seeing their password," typically involving an authorization server issuing access/refresh tokens. FastAPI's `OAuth2PasswordBearer` is a *transport convention*, not full OAuth2 — for TaskFlow's own login, you'll implement a password-grant-like flow that issues JWTs, which is simpler than full OAuth2 but follows the same header conventions (`Authorization: Bearer <token>`).

**Common pitfalls:**
- Storing JWTs in `localStorage` on the frontend (vulnerable to XSS) instead of an `HttpOnly` cookie.
- Forgetting to set and check `exp` (expiry) — a JWT with no expiry is a permanent credential if leaked.
- Conflating "valid token" with "authorized for this resource" — a logged-in user is authenticated, but you still need a separate check (e.g., "is this the task's owner?") for authorization.
- Rolling your own crypto/signing. Use a maintained library (`python-jose`, `pyjwt`) and a well-known algorithm (HS256 for a shared-secret setup, RS256 if you need asymmetric verification).

```python
from jose import jwt
from datetime import datetime, timedelta, timezone

def create_access_token(subject: str, secret: str, expires_minutes: int = 30) -> str:
    expire = datetime.now(timezone.utc) + timedelta(minutes=expires_minutes)
    return jwt.encode({"sub": subject, "exp": expire}, secret, algorithm="HS256")
```

## API documentation: OpenAPI/Swagger auto-generation

FastAPI generates an OpenAPI schema from your route signatures, Pydantic models, and docstrings, and serves interactive docs at `/docs` (Swagger UI) and `/redoc`. This is one of FastAPI's biggest wins: documentation that cannot drift from the implementation because it's *derived* from it.

**Why it matters:** hand-written API docs rot the moment someone changes an endpoint and forgets to update the wiki page. Auto-generated docs are always current because they're generated from the same source of truth as the code.

**Common pitfalls:**
- Auto-generation is only as good as your type hints and models — an endpoint typed to return `dict` documents nothing useful. Always give response models (`response_model=TaskRead`).
- Forgetting to document error responses — FastAPI won't automatically know your endpoint can return a `404`; use `responses={404: {"model": ErrorResponse}}` to document it.
- Leaving default/example values out of Pydantic models makes the generated docs harder for consumers to use — add `json_schema_extra` or `Field(examples=[...])`.

**Do This** in this chapter includes generating the docs and reviewing them critically for misleading or missing information — treat this as a real code-review step, not a formality.

## Error handling & consistent error response design

Every endpoint should fail in a predictable *shape*. Without a convention, some endpoints return `{"error": "..."}`,  others `{"detail": "..."}`, others a raw stack trace — and every API consumer has to special-case each one.

A good pattern: a single `ErrorResponse` model (e.g., `{"code": "TASK_NOT_FOUND", "message": "...", "details": {...}}`) and a global exception handler that converts internal exceptions into this shape.

```python
from fastapi import Request
from fastapi.responses import JSONResponse

@app.exception_handler(TaskNotFoundError)
async def task_not_found_handler(request: Request, exc: TaskNotFoundError):
    return JSONResponse(status_code=404, content={"code": "TASK_NOT_FOUND", "message": str(exc)})
```

**Common pitfalls:**
- Leaking internal exception messages (or stack traces) to clients in production — a great way to expose implementation details or SQL fragments.
- Returning `200 OK` with an error payload in the body ("soft 200s") — this defeats HTTP semantics and breaks tooling/monitoring that keys off status codes.
- Not distinguishing client errors (4xx, don't retry blindly) from server errors (5xx, safe to retry with backoff) in your response contract.

## Rate limiting, pagination, filtering conventions

**Rate limiting** protects your service from being overwhelmed (accidentally or maliciously). Common approaches: token bucket or sliding-window counters, often backed by Redis (which TaskFlow already has from Chapter 5) since it needs to be shared across server instances. Respond with `429 Too Many Requests` and a `Retry-After` header.

**Pagination** prevents an endpoint like `GET /tasks` from returning millions of rows in one response.
- **Offset/limit** (`?offset=20&limit=10`) — simple, but degrades in performance on large tables and is unstable if rows are inserted/deleted between pages.
- **Cursor-based** (`?cursor=<opaque-token>`) — stable under concurrent writes, scales better, but harder to implement and doesn't support "jump to page 5" UIs.

**Filtering** conventions (`?status=done&priority=3`) should be documented, validated (don't accept arbitrary field names blindly — that risks exposing internal column names or enabling injection through dynamic query building), and combined predictably with pagination (filter first, then paginate).

**Common pitfalls:**
- Unbounded `limit` values that let a client request a million rows in one call — always cap it.
- Rate limiting only at the load balancer and not being able to give per-user limits when you need them (e.g., stricter limits for anonymous vs authenticated users).
- Silently ignoring unknown filter/query params instead of returning a `400` — silent ignoring hides client bugs.
