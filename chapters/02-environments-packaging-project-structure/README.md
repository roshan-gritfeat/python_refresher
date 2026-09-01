# Chapter 2 — Environments, Packaging & Project Structure

**Goal:** stop scattering `.py` files in a folder and start building installable, shareable packages — the line between "a script I run" and "software other people can depend on."

---

## Virtual Environments

A virtual environment is an isolated Python installation (interpreter + its own `site-packages`) so that one project's dependencies don't collide with another's, or with the system Python. Without one, installing package `X==1.0` for project A and `X==2.0` for project B on the same machine is impossible without conflict.

- **`venv`** — built into the standard library (`python -m venv .venv`). No extra install needed, works everywhere, but dependency resolution and locking are manual (paired with `pip` + `requirements.txt`).
- **`uv`** — a modern, Rust-based tool that handles venv creation, dependency resolution, and locking, and is dramatically faster than `pip`. Increasingly the default recommendation for new projects in 2024+.
- **`poetry`** — an all-in-one dependency manager and build tool with its own lock file (`poetry.lock`) and `pyproject.toml`-based configuration; strong for teams wanting one consistent tool for env + packaging + publishing.
- **`pipenv`** — an earlier attempt to combine `pip` + virtualenv management with a `Pipfile`/`Pipfile.lock`; largely superseded by `poetry`/`uv` for new projects, but still seen in legacy codebases.

**Trade-offs**: `venv` is the "always available, zero magic" baseline everyone should understand first. `uv`/`poetry` add speed and reproducibility but are another tool to install and another config format to learn. The pragmatic default in 2025+ is `uv` for new projects and plain `venv` for understanding what's happening underneath.

**Pitfall**: forgetting which environment is active. Installing a package while the wrong venv (or no venv, i.e. the system Python) is active is one of the most common "works on my machine" bugs in the wild.

---

## Dependency Management

- **`requirements.txt`** — a flat list of package names/versions, historically installed via `pip install -r requirements.txt`. Simple, but has no standard way to distinguish "this project's declared dependencies" from "the fully resolved set of exact versions that were actually installed" unless you maintain separate files by convention (`requirements.txt` vs `requirements-lock.txt`).
- **`pyproject.toml`** — the modern, standardized (PEP 517/518/621) place to declare a project's metadata and dependencies in one file, replacing the old patchwork of `setup.py`/`setup.cfg`/`requirements.txt`.
- **Lock files** (`poetry.lock`, `uv.lock`, `Pipfile.lock`) pin the *exact* resolved versions (including transitive dependencies) so that "it works on my machine" reliably becomes "it works on every machine," because everyone installs from the identical, fully-resolved dependency graph — not just compatible ranges.
- **Semantic versioning (SemVer)** — `MAJOR.MINOR.PATCH`. A major bump signals breaking changes, minor adds backward-compatible features, patch is a backward-compatible bug fix. Understanding SemVer is what lets you safely write a dependency constraint like `>=2.1,<3.0` instead of pinning an exact version and never updating.
- **Dependency pinning** — deciding how tightly to constrain versions. Too loose (`requests` with no version at all) risks a future breaking release silently breaking your app; too tight (`requests==2.28.1` forever) accumulates security debt and blocks upgrades. Applications (which you deploy) typically pin tightly via a lock file; libraries (which others depend on) typically specify loose, compatible ranges to avoid "dependency hell" for their consumers.

---

## Project Layout: `src` Layout vs Flat Layout

**Flat layout** puts your package directly at the repository root:

```
myproject/
    mypackage/
        __init__.py
        core.py
    tests/
    pyproject.toml
```

**`src` layout** nests the package one level deeper:

```
myproject/
    src/
        mypackage/
            __init__.py
            core.py
    tests/
    pyproject.toml
```

**Why `src` layout avoids import bugs**: with a flat layout, running Python from the repository root can accidentally import your package directly from the source tree (because the current directory is implicitly on `sys.path`) even if it was never properly installed. This masks packaging bugs — code appears to work in development but fails for a user who `pip install`s it, because the *installed* version and the *editable source-tree* version can silently diverge, or a bad `pyproject.toml` (missing files, wrong package discovery) goes unnoticed. The `src` layout forces you to actually install the package (even in editable mode, `pip install -e .`) before you can import it at all, which means what you test locally is guaranteed to be the same thing that gets published and installed elsewhere.

---

## Packaging

**`pyproject.toml`** is the single, standardized configuration file introduced across three PEPs:
- **PEP 517** defines a standard interface between build tools and frontends (so `pip` doesn't need to know whether you use `setuptools`, `hatchling`, `flit`, or `poetry-core` under the hood).
- **PEP 518** defines the `[build-system]` table, declaring what's needed to build your package before it's even installed.
- **PEP 621** standardizes project metadata (name, version, dependencies, authors) directly in `[project]`, replacing the old `setup.py`/`setup.cfg` split.

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "taskflow"
version = "0.1.0"
description = "A task tracker CLI"
requires-python = ">=3.10"
dependencies = []

[project.scripts]
taskflow = "taskflow.cli:main"
```

**Building wheels**: `python -m build` produces a `.whl` (wheel — a pre-built, fast-to-install binary-ish distribution format) and an `.sdist` (source distribution). Wheels are what `pip install somepackage` downloads and installs in the common case — no build step needed at install time.

**Publishing to PyPI** (or a private registry, e.g., an internal Artifactory/Nexus or GitHub Packages) is done via `twine upload dist/*` after building. **TestPyPI** (`test.pypi.org`) is a free sandbox instance of PyPI for practicing the publish flow without polluting the real index or squatting a real package name.

---

## Entry Points and CLI Tools

The `[project.scripts]` table in `pyproject.toml` (the modern `console_scripts` mechanism) lets your package install an actual command-line executable:

```toml
[project.scripts]
taskflow = "taskflow.cli:main"
```

After `pip install .`, the user can run `taskflow` directly from their shell, and it dispatches to the `main()` function in `taskflow/cli.py` — no need to remember `python -m taskflow` or a script path. This is what turns "a Python file with a `if __name__ == '__main__':` block" into "an actual installed tool," and it's central to the chapter's "hand your project's zip file to a stranger" readiness bar.

---

## Configuration Management

- **`.env` files** store environment-specific values (API keys, database URLs, debug flags) outside of source code, loaded at runtime (commonly via `python-dotenv`). `.env` must always be in `.gitignore` — committing it is a classic way secrets end up in git history forever.
- **`pydantic-settings`** provides typed, validated configuration objects that read from environment variables (and `.env` files) with sensible defaults and automatic type coercion/validation — catching a missing or malformed config value at startup instead of deep inside a request handler.

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    debug: bool = False

    class Config:
        env_file = ".env"

settings = Settings()
```

- **Avoiding hardcoded secrets**: never write API keys, passwords, or tokens directly into source code — not even "temporarily." They end up in git history, log output, and error tracebacks. Use environment variables/secret managers, and scan for accidental leaks (Chapter 11 covers tooling for this in depth).

---

## Python Versioning Strategy

- **`pyenv`** lets you install and switch between multiple Python versions on one machine (e.g., testing your package against 3.10, 3.11, 3.12) without touching the system Python.
- **Supporting multiple Python versions**: declare the range your package supports via `requires-python` in `pyproject.toml`, and actually test against the oldest and newest versions you claim to support (a matrix, formalized properly in Chapter 9's CI) — claiming support you haven't verified is a common source of "works for me" bug reports from users on older interpreters.
- A sound strategy: support the currently-maintained upstream Python versions (check the official end-of-life schedule), drop end-of-life versions promptly, and avoid using brand-new syntax features until they've been available for at least one prior minor release, to keep the adoption curve smooth for your users.

---

## Do This — Hands-On Tasks

1. Take your Chapter 1 script and turn it into a proper installable package using `src` layout with a `pyproject.toml` (PEP 621 metadata, a build backend, and correct package discovery).
2. Publish it to TestPyPI (free, safe sandbox) — this is a stretch/optional goal if you don't want a public account trail, but attempt the full `build` + `twine upload --repository testpypi` flow at least once in your Python career.
3. Add a CLI entry point (`[project.scripts]`) so the tool can be run as `taskflow --help` after installation, with no need to invoke `python` directly.

## You'll Know You're Ready When...

You can hand your project's zip file (or a `git clone` URL) to a stranger, and they can `pip install .` and run it with zero manual fixes — no "oh you also need to set this environment variable" or "oh you need to `cd` into the right folder first" caveats.
