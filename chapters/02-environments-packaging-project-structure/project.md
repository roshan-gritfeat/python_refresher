# TaskFlow — Chapter 2 Increment

## Where TaskFlow stands

After Chapter 1, TaskFlow is idiomatic Python: a `@dataclass`-based `Task`, generator-based filtering/listing, `@timed` and `@retry` decorators, a `TaskFileSession` context manager for persistence, custom exceptions used with EAFP, and full type hints verified by `mypy`/`pyright`. It is still one (or a couple of) loose `.py` file(s) with no formal packaging.

## What to add this chapter

Turn TaskFlow into a proper, installable Python package using `src` layout, per the syllabus's "Do This."

1. **Restructure into `src` layout**:
   ```
   taskflow/
       src/
           taskflow/
               __init__.py
               models.py       # Task dataclass
               storage.py      # TaskFileSession, save/load, pathlib-based persistence
               exceptions.py   # TaskNotFoundError, etc.
               decorators.py   # @timed, @retry
               cli.py          # command loop / argument parsing, main() entry point
       tests/                  # empty or placeholder for now — real tests arrive in Chapter 3
       pyproject.toml
       README.md
       .gitignore
       .env.example
   ```
2. **Write `pyproject.toml`** with PEP 621 metadata: `name = "taskflow"`, a version, `requires-python`, and a `[build-system]` using a modern backend (`hatchling` or `setuptools`). Split any external dependencies you've picked up (e.g., `pydantic-settings` for config) into `[project.dependencies]`.
3. **Add a CLI entry point** via `[project.scripts]`: `taskflow = "taskflow.cli:main"`, so after `pip install -e .`, the command `taskflow` works directly, replacing `python task_tracker.py`.
4. **Introduce configuration management** — add a `Settings` class (via `pydantic-settings`) that reads the task-storage file path (and any other configurable value, like default priority) from environment variables / a `.env` file, with a sensible default so the tool still works with zero configuration. Ship a `.env.example` (never a real `.env`) documenting available settings, and ensure `.env` is in `.gitignore`.
5. **Verify the editable install loop** — `pip install -e .` in a fresh venv, then run `taskflow add "test"`, `taskflow list`, etc., confirming everything works purely from the installed console script, not from running a script file directly.
6. **(Stretch) Publish to TestPyPI** — build with `python -m build`, upload with `twine upload --repository testpypi dist/*`, then `pip install --index-url https://test.pypi.org/simple/ taskflow` into a brand-new venv and confirm the CLI works end-to-end from a public (sandbox) index.
7. **Pin a Python version strategy** — set `requires-python = ">=3.10"` (or your actual floor), and note in the README which versions you've actually tested against.

## Concrete tasks checklist

- [ ] Package restructured into `src/taskflow/` with logical module separation (models, storage, exceptions, decorators, cli).
- [ ] `pyproject.toml` written with correct `[build-system]`, `[project]` metadata, and dependencies.
- [ ] `[project.scripts]` entry point added; `taskflow --help` (or a basic help output) works after `pip install -e .`.
- [ ] `pydantic-settings`-based `Settings` class added for configurable storage path/defaults, backed by `.env` support.
- [ ] `.env.example` committed; real `.env` (if created for local testing) is gitignored.
- [ ] Fresh-venv install-and-run verified from scratch (delete venv, recreate, `pip install -e .`, run the CLI).
- [ ] (Stretch) Package built and uploaded to TestPyPI, and installed back down from there into a clean venv.

## Definition of done

- [ ] A stranger (or you, in a brand-new venv on a different machine/folder) can run `pip install .` (or `pip install -e .` for development) against the repository and immediately run `taskflow` with zero manual fixes — no missing files, no hardcoded paths, no "oh you also need to..." caveats.
- [ ] All prior functionality (add/complete/remove/list/find tasks, JSON persistence, decorators, custom exceptions) works identically through the installed CLI command.
- [ ] `pyproject.toml` is the single source of project metadata and dependencies — no stray `requirements.txt` duplicating the same information (a lock file is fine and expected).
- [ ] Secrets/config are never hardcoded; all configurable values flow through the `Settings` class and environment variables.
