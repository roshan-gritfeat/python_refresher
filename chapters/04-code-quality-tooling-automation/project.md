# TaskFlow — Chapter 4 Increment: Code Quality, Tooling & Automation

## Where TaskFlow stands

After Chapter 3, TaskFlow has a `tests/` directory with unit and integration pytest tests, 3+ fixtures in `conftest.py`, parametrized tests, a confirmed bug-catch, and one feature built via TDD. It still has no automated linting, type checking, formatting, or commit gating — quality currently depends entirely on the developer remembering to run `pytest`.

## What to add this chapter

1. **Linting & formatting with ruff.**
   - Add `ruff` as a dev dependency.
   - Add a `[tool.ruff]` / `[tool.ruff.lint]` / `[tool.ruff.format]` section to `pyproject.toml`, enabling at minimum pyflakes (`F`), pycodestyle (`E`/`W`), isort (`I`), and bugbear (`B`).
   - Run `ruff check --fix .` and `ruff format .` across the whole `src/` and `tests/` trees, fixing everything that surfaces, and commit the cleaned-up code as its own commit.

2. **Static typing with mypy.**
   - Add `mypy` as a dev dependency and a `[tool.mypy]` section in `pyproject.toml`.
   - Start permissive at the project level, then add a `[[tool.mypy.overrides]]` block that enables `disallow_untyped_defs = true` for TaskFlow's core domain module(s) (the `Task` model and storage layer) first.
   - Fix every mypy error surfaced in the strict-checked modules — this may mean adding missing annotations or fixing genuine type mismatches (e.g., a function that can return `None` but wasn't typed `Optional`).

3. **Pre-commit hooks.**
   - Add a `.pre-commit-config.yaml` wiring up: `ruff` (lint, with `--fix`), `ruff-format`, `mypy`, and at least one general hygiene hook from `pre-commit/pre-commit-hooks` (e.g. `trailing-whitespace`, `end-of-file-fixer`, `check-added-large-files`).
   - Run `pre-commit install` and verify with a real `git commit` attempt that a deliberately bad change (an unformatted file, or a new mypy error) is blocked.
   - Add a note to the project README on how a new contributor sets up pre-commit (`pip install pre-commit && pre-commit install`).

4. **Docstrings and generated docs.**
   - Write complete Google-style (or NumPy-style — pick one and be consistent) docstrings for every public class and function in TaskFlow's package (the `Task` model, the storage/repository interface, and the CLI commands).
   - Set up `mkdocs` with `mkdocstrings` (or Sphinx with `autodoc` + `napoleon`) to auto-generate an API reference site from those docstrings, and confirm it builds without warnings.

5. **Task runner.**
   - Add a `justfile` (or `Makefile`, or `nox`/`tox` config) with at minimum `test`, `lint`, `typecheck`, `format`, and `check` (running lint + typecheck + test together) targets, so quality gates are one command.

6. **Self code-review exercise.**
   - Pick the TaskFlow code you wrote in Chapter 2 or 3 (at least a week old relative to today) and write a structured self-review as `docs/self-review-ch4.md` (or similar), listing at least 5 concrete issues found — correctness, missing tests, and design, not just style (style issues should already be caught by ruff at this point).
   - Fix at least 2 of the issues found and note which ones you deliberately deferred and why.

## Definition of done

- [ ] `ruff`, `mypy`, `pre-commit` added as dev dependencies; configs live in `pyproject.toml` and `.pre-commit-config.yaml`.
- [ ] `ruff check .` and `ruff format --check .` both pass with zero issues across the whole repo.
- [ ] `mypy` passes with zero errors, with strict mode enabled on at least the core domain module(s).
- [ ] `pre-commit install` is set up and demonstrably blocks a bad commit (documented or screenshotted in the self-review note).
- [ ] Every public function/class in TaskFlow has a complete, non-trivial docstring.
- [ ] A generated docs site (mkdocs or Sphinx) builds cleanly from those docstrings.
- [ ] A task runner file (justfile/Makefile/noxfile) exposes `test`, `lint`, `typecheck`, and `check` commands.
- [ ] A self-review document lists 5+ real issues found in week-old code, with at least 2 fixed.
