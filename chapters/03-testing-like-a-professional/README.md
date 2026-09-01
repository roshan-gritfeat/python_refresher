# Chapter 3 — Testing Like a Professional

Goal: tests aren't an afterthought — they're how you prove your code works and keep working.

---

## Testing Philosophy: Unit vs Integration vs End-to-End, the Testing Pyramid

Professional software is tested at multiple levels, and understanding the differences is the difference between a suite that gives you confidence and one that just makes CI slow.

**Unit tests** exercise a single function, method, or class in isolation, with all external dependencies (databases, network calls, filesystems, other modules) either absent or replaced with test doubles. They are fast (milliseconds), numerous (hundreds to thousands per project), and pinpoint failures precisely — when a unit test fails, you know almost exactly which line of logic broke. Their weakness is that they can't tell you whether the pieces work *together*.

**Integration tests** verify that two or more real components cooperate correctly — for example, your repository layer talking to an actual SQLite database, or your CLI parser invoking your business logic. They are slower and less numerous than unit tests, but they catch the class of bugs unit tests are blind to: wrong SQL, mismatched function signatures across modules, serialization mismatches, misconfigured wiring.

**End-to-end (E2E) tests** drive the system the way a real user or client would — spawning the actual CLI process, hitting a real (or realistic) API, and observing final output. They give the highest confidence that the *product* works, but they are slow, flaky (network, timing, environment-dependent), and expensive to maintain, so you want very few of them.

The **testing pyramid** is the classic mental model: many unit tests at the base, a moderate number of integration tests in the middle, and a thin layer of E2E tests at the top. The shape matters because of a cost/confidence tradeoff — unit tests are cheap and fast but low-confidence about the whole system; E2E tests are high-confidence but expensive and slow. Invert the pyramid (lots of E2E, few units) and your CI takes 40 minutes and flakes constantly; get the pyramid right and you get fast feedback with reasonable confidence.

**Common pitfall:** treating "coverage" as a synonym for "quality" and writing a thousand trivial unit tests that assert getters return what you set, while leaving the actual business rules (the parts that break in production) untested. Write tests that would fail if the behavior were wrong, not tests that merely execute the code.

```
        /  E2E  \        <- few, slow, high confidence
       /----------\
      / Integration \    <- some, moderate speed
     /----------------\
    /   Unit Tests      \ <- many, fast, isolated
   /------------------------\
```

---

## pytest Fundamentals: Fixtures, Parametrize, Marks, conftest.py

`pytest` is the de facto standard Python test runner because it lets you write tests as plain functions with `assert` statements — no `self.assertEqual` boilerplate — while still providing powerful mechanisms for setup/teardown and test generation.

**Fixtures** are pytest's dependency-injection mechanism for test setup. A function decorated with `@pytest.fixture` produces a value (or performs setup/teardown around a `yield`) that any test can request simply by naming it as a parameter. Fixtures replace `setUp`/`tearDown` from `unittest` with something more composable: fixtures can depend on other fixtures, and they have configurable **scope** (`function` — default, re-run per test; `class`; `module`; `session` — run once per whole test run). Use narrow scope for anything mutable (e.g., a fresh in-memory database per test) and broad scope for expensive, immutable setup (e.g., a compiled schema).

```python
import pytest
from taskflow.storage import InMemoryTaskStore

@pytest.fixture
def store():
    """A fresh, empty task store for each test."""
    return InMemoryTaskStore()

@pytest.fixture
def populated_store(store):
    store.add("Buy milk")
    store.add("Write report")
    return store

def test_add_increases_count(store):
    store.add("Buy milk")
    assert len(store.all()) == 1

def test_populated_store_has_two_tasks(populated_store):
    assert len(populated_store.all()) == 2
```

**Parametrize** (`@pytest.mark.parametrize`) lets one test function run against many input/output pairs, avoiding copy-pasted near-duplicate tests and making the input space explicit and easy to extend.

```python
@pytest.mark.parametrize("raw,expected", [
    ("", False),
    ("   ", False),
    ("Buy milk", True),
    ("x" * 300, False),  # too long
])
def test_is_valid_title(raw, expected):
    assert is_valid_title(raw) is expected
```

**Marks** (`@pytest.mark.slow`, `@pytest.mark.skip`, `@pytest.mark.xfail`, or custom marks registered in `pyproject.toml`) let you tag tests for selective running (`pytest -m "not slow"`), mark known failures, or skip tests conditionally (`@pytest.mark.skipif(sys.platform == "win32", reason=...)`).

**`conftest.py`** is a special file pytest auto-discovers; fixtures (and hooks) defined there are available to every test file in that directory and below, without imports. This is how you share fixtures across a whole test suite without a fixture-import free-for-all. A common pitfall is putting too much unrelated setup into a single top-level `conftest.py`, making it unclear which fixtures a given test file actually depends on — prefer several smaller, well-named fixtures over one mega-fixture.

**Pitfalls:** fixture scope bugs (a `session`-scoped fixture that's accidentally mutated by one test leaks state into others); fixtures that hide too much test setup, making tests unreadable independent of the fixture's definition; and over-parametrizing until the test's *intent* is buried in a wall of tuples.

---

## Mocking & Patching: unittest.mock, When Mocking Is a Smell vs Necessary

`unittest.mock` lets you replace real objects with fake stand-ins (`Mock`, `MagicMock`) that record how they were called and can be configured to return canned values or raise exceptions. `patch` (as a decorator, context manager, or via `monkeypatch` in pytest) temporarily replaces an attribute/object for the duration of a test, then restores it automatically.

Mocking is **necessary** when a real dependency is slow, non-deterministic, costly, or has side effects you don't want in a test — an external HTTP API, sending a real email, the system clock, or a paid third-party service. It's also useful for testing error paths that are hard to trigger for real (e.g., "the network call times out").

```python
from unittest.mock import patch

def test_sync_uses_current_time(monkeypatch):
    fixed = "2026-08-31T00:00:00"
    monkeypatch.setattr("taskflow.clock.now_iso", lambda: fixed)
    task = create_task("Ship report")
    assert task.created_at == fixed
```

Mocking is a **smell** when it's used to avoid testing your own code's real logic — e.g., mocking your own repository class inside a test of that same repository, or mocking so many collaborators that the test only proves "the mocks were called," not that the system behaves correctly. A common failure mode is a test suite that's 100% green after a refactor breaks production, because every test mocked out the very component that changed. The rule of thumb: mock at the boundary of your system (external I/O, third-party services, time, randomness), not the internals you're trying to verify. Prefer integration tests with a real (in-memory or throwaway) dependency over mocking your own database layer, when feasible — a real SQLite file or `:memory:` database is often cheap enough to just use.

---

## Test Coverage: coverage.py, What % Coverage Actually Means (and Its Limits)

`coverage.py` (usually run via `pytest-cov`) instruments your code to report which lines (and optionally branches) executed during the test run. A coverage report is a *diagnostic tool for finding untested code*, not a quality score.

**What it tells you:** which lines never ran under any test — a strong signal that a bug there would go unnoticed.

**What it does NOT tell you:** whether the assertions in your tests are meaningful. A test that calls a function and asserts nothing (or asserts something trivially true) gives 100% coverage of that function while proving nothing about correctness. Coverage also can't detect missing test *cases* for logic that a single line covers — e.g., one line `return a / b` is "covered" by a single passing test, but a `ZeroDivisionError` edge case might be completely untested even though every line shows green.

Use coverage as a floor, not a target: enforce a reasonable minimum (e.g., 80-90%) to catch obviously untested modules, but treat "100% coverage" claims with skepticism, and always pair coverage numbers with a mental (or documented) review of *which behaviors* are tested, not just which lines executed. Branch coverage (`--branch` in coverage.py / `--cov-branch`) is stronger than line coverage because it also checks that both sides of an `if` were exercised.

---

## Test-Driven Development (TDD): Red-Green-Refactor Cycle

TDD is a discipline, not a tool: you write a failing test *before* writing the implementation, then write the minimum code to make it pass, then refactor with the safety net of that passing test.

1. **Red** — write a test for behavior that doesn't exist yet; run it and watch it fail (for the *right* reason — a missing function, not a typo).
2. **Green** — write the simplest possible code that makes the test pass, resisting the urge to over-engineer.
3. **Refactor** — clean up the implementation (and/or the test) now that you have a safety net proving the behavior still works.

The value of TDD isn't ceremony — it's that it forces you to think about the *interface and contract* of a function before you think about its implementation, and it guarantees every piece of behavior has at least one test, because the test was written first. A common pitfall is writing a test so tied to implementation details (e.g., asserting a private helper was called a specific number of times) that any refactor breaks the test even though behavior didn't change — prefer testing observable behavior (inputs → outputs / state changes) over internal mechanics.

---

## Property-Based Testing (hypothesis)

Traditional example-based tests assert specific input/output pairs you thought of. **Property-based testing**, via the `hypothesis` library, instead asserts a *property* that should hold for a whole class of inputs, and the library generates hundreds of varied (including adversarial edge-case) inputs to try to break it.

```python
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_sorting_is_idempotent(xs):
    once = sorted(xs)
    twice = sorted(once)
    assert once == twice

@given(st.text())
def test_slug_is_always_lowercase(s):
    assert slugify(s) == slugify(s).lower()
```

This is especially powerful for parsing, serialization round-trips (`decode(encode(x)) == x`), mathematical invariants, and anything with a large or awkward input space where you'd otherwise have to guess edge cases by hand. The pitfall is spending too much effort writing exhaustive properties for simple logic where a couple of example tests would do — reserve hypothesis for functions where the input space is genuinely large or edge cases are easy to miss (e.g., string parsing, date arithmetic, numeric algorithms).

---

## Testing Edge Cases: Exceptions, Boundary Values, Async Code

**Exceptions:** every function that can raise should have a test proving it raises the *right* exception under the *right* condition, using `pytest.raises`:

```python
def test_add_rejects_empty_title():
    with pytest.raises(ValueError, match="title cannot be empty"):
        create_task("")
```

**Boundary values:** off-by-one errors cluster at boundaries — empty collections, single-element collections, the maximum allowed length, zero, negative numbers, the first/last item in a loop. Explicitly test `n=0`, `n=1`, and `n=max` rather than only "typical" values.

**Async code:** testing `async def` functions requires an async-aware test runner — `pytest-asyncio` (mark tests `@pytest.mark.asyncio`) or `anyio`'s pytest plugin. Common pitfalls: forgetting to `await` inside the test (silently returns a coroutine object instead of running it — pytest-asyncio strict mode helps catch this), and mocking async functions with a plain `Mock` instead of `AsyncMock`, which produces a coroutine-shaped object that doesn't behave like the original.

---

## Continuous Testing in CI (Preview)

Running `pytest` locally proves the code works *on your machine, right now*. Continuous Integration (full treatment in Chapter 9) runs your test suite automatically on every push/PR, on a clean environment, so regressions are caught before they reach `main` and before another developer pulls broken code. Even before building a full CI pipeline, get in the habit of running the *entire* suite (not just the tests you think are relevant) before every commit, and keep the suite fast enough that doing so is not a chore — this is what makes CI painless to add later rather than a battle against a slow, flaky suite.
