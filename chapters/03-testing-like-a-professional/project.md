# TaskFlow — Chapter 3 Increment: Testing Like a Professional

## Where TaskFlow stands

By the end of Chapter 2, TaskFlow is an installable `src`-layout Python package with a `pyproject.toml`, using dataclasses, type hints, decorators, and context managers internally, and exposing a `taskflow` CLI entry point (via `console_scripts`) for adding, listing, completing, and removing tasks. It has no automated tests yet.

## What to add this chapter

1. **Test infrastructure.**
   - Add `pytest`, `pytest-cov`, `pytest-asyncio`, and `hypothesis` as dev dependencies in `pyproject.toml` (an `[project.optional-dependencies]` group named `dev` or `test`, or a `dependency-groups` table).
   - Create a `tests/` directory at the project root (parallel to `src/`), mirroring the package's internal module structure (e.g., `tests/test_models.py`, `tests/test_storage.py`, `tests/test_cli.py`).
   - Add a `tests/conftest.py` with at least **3 shared fixtures**, for example:
     - `empty_store` — a fresh in-memory/temp-file task store.
     - `populated_store` — a store pre-loaded with a handful of tasks in varied states (done/not done, different priorities).
     - `cli_runner` — a helper (e.g. wrapping `click.testing.CliRunner` if using Click, or `subprocess`/direct function calls otherwise) for invoking the CLI in tests.
   - Configure `pytest` in `pyproject.toml` ([tool.pytest.ini_options]) with test paths and any custom marks, and configure `coverage.py` ([tool.coverage.run]/[tool.coverage.report]) with a minimum threshold.

2. **Unit tests for the core domain.**
   - Test the `Task` dataclass: construction, default values, `__eq__`/`__repr__` behavior, any validation in `__post_init__`.
   - Test the storage/repository layer's add, list, complete, delete, and filter operations against the `empty_store`/`populated_store` fixtures.
   - Use `@pytest.mark.parametrize` for at least one validation function (e.g., title length limits, priority enum values) covering multiple valid/invalid inputs in one test.

3. **CLI tests (integration level).**
   - Test that invoking the CLI's `add`, `list`, `done`, and `remove` commands produces the expected output and side effects, using the `cli_runner` fixture — these are integration tests since they exercise CLI parsing + business logic + storage together.

4. **Deliberate bug and detection.**
   - Introduce one deliberate bug into TaskFlow's logic (e.g., an off-by-one in pagination, or a `complete_task` that doesn't persist the change, or a mutable-default-argument bug in a task-creation helper).
   - Run the full test suite and confirm at least one test fails because of it, and record (in a code comment or a short note in this file) which test caught it and why.
   - Fix the bug and confirm the suite goes green again.

5. **TDD a new feature.**
   - Pick one small new feature not yet in TaskFlow (e.g., "due dates" or "tags/labels" on a task, or a `taskflow overdue` command).
   - Follow red-green-refactor explicitly: write the failing test(s) first, implement the minimum to pass, then refactor.
   - Keep the failing-test-first commit (or at least document the sequence) so the TDD process is visible.

6. **Mocking where it's warranted.**
   - If TaskFlow's storage reads the system clock (e.g., `created_at` timestamps) or the filesystem, add at least one test that mocks/patches that dependency (`monkeypatch` or `unittest.mock.patch`) so the test is deterministic.

7. **(Optional stretch) Property-based test.**
   - Add one `hypothesis`-based property test for a pure function in TaskFlow (e.g., a slugify/ID-generation helper, or sort/filter logic) asserting an invariant across generated inputs.

## Definition of done

- [ ] `pytest`, `pytest-cov`, `pytest-asyncio`, `hypothesis` declared as dev/test dependencies in `pyproject.toml`.
- [ ] `tests/` directory exists with a `conftest.py` containing at least 3 fixtures.
- [ ] At least one test uses `@pytest.mark.parametrize` with 3+ cases.
- [ ] Unit tests cover the `Task` model and the storage/repository layer.
- [ ] At least one integration-level test exercises the CLI end-to-end via the test fixture/runner.
- [ ] A deliberate bug was introduced, confirmed to fail a specific test, then fixed — documented in a comment or note.
- [ ] One new feature was built via explicit red-green-refactor TDD.
- [ ] At least one test uses mocking/patching for a non-deterministic dependency (time, filesystem, or network).
- [ ] `pytest --cov` runs clean and coverage is at or above the threshold configured in `pyproject.toml`.
- [ ] Full suite passes with a single `pytest` invocation from the project root.
