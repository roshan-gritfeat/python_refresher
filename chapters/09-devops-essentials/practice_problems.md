# Chapter 9 — Practice Problems: DevOps Essentials

Standalone exercises, ordered easy to hard. None of these touch TaskFlow directly — build small throwaway apps/scripts to drill each concept.

1. **Write a Dockerfile for a trivial script.** Containerize a single `hello.py` script that prints the Python version.
   *Focus: `FROM`, `WORKDIR`, `COPY`, `CMD` basics.*

2. **Add a `.dockerignore`.** Take a folder containing `.venv/`, `__pycache__/`, a `.env` file, and a `.git/` directory, and write a `.dockerignore` that excludes all of them from a build context.
   *Focus: understanding what actually gets sent to the Docker daemon as "build context."*

3. **Shrink an image.** Given a Dockerfile using `python:3.12` (full image) that installs one pure-Python dependency, switch it to `python:3.12-slim` and compare image sizes with `docker images`.
   *Hint: use `docker image ls` before/after; note what changes and what could break.*

4. **Convert a single-stage Dockerfile with a C-extension dependency (e.g., `psycopg2` or `cryptography`) into a multi-stage build.**
   *Focus: builder stage installs build tools and compiles; final stage copies only the installed packages.*

5. **Run as a non-root user.** Take any working Dockerfile and modify it so the container process runs as a created non-root user instead of root. Verify with `docker run <image> whoami`.
   *Hint: `RUN useradd` + `USER` instruction, and check file permissions on copied files.*

6. **Write a docker-compose file for a two-service toy app** (a simple Flask/FastAPI app + a Redis cache) where the app increments and reads a counter stored in Redis.
   *Focus: service networking — connecting to Redis by service name, not `localhost`.*

7. **Add a healthcheck.** Extend the compose file from #6 so the app service only starts after Redis reports healthy, using `depends_on: condition: service_healthy`.
   *Hint: Redis's official image supports `redis-cli ping` as a healthcheck test.*

8. **Write a GitHub Actions workflow that runs `ruff check` and `pytest` on every push**, for a small existing Python package of your choice (or a toy one you create).
   *Focus: job structure, `actions/setup-python`, caching pip dependencies with `actions/cache` or `setup-python`'s built-in cache.*

9. **Add a `needs:` dependency chain** so a hypothetical "build" job in your workflow only runs if both lint and test jobs succeeded, and only on pushes to `main` (not pull requests).
   *Hint: `needs: [lint, test]` plus an `if:` condition referencing `github.ref` and `github.event_name`.*

10. **Simulate environment-based configuration.** Write a small script that reads a `DATABASE_URL` environment variable via `pydantic-settings` and connects to a different (mock) target depending on whether the variable is unset (defaults to local SQLite) or set (points elsewhere) — without any `if ENV == "prod"` branching in application code.
    *Focus: environment parity — same code path, different config.*

11. **Deploy a "hello world" FastAPI app to a free-tier PaaS** (Render, Railway, or Fly.io) directly from a GitHub repo, and hit the deployed URL from a browser or `curl`.
    *Hint: most PaaS platforms auto-detect a Dockerfile; check their dashboard logs when a deploy fails.*

12. **Simulate a secret leak and fix it.** Commit a file containing a fake API key to a local git repo, then (a) add it to `.gitignore` going forward, and (b) research and describe (without necessarily executing) what it would take to fully remove it from git history — e.g., `git filter-repo` — and why rotating the real secret is still mandatory even after that.
    *Focus: understanding that removing a file in a new commit does not erase it from history.*
