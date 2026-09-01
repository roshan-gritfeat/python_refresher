# Chapter 1 — Practice Problems

Standalone exercises to drill idiomatic Python. Ordered easy to hard. No solutions provided.

1. **EAFP refactor** — Take a LBYL-style function (nested `if key in dict` / `if os.path.exists` checks) and rewrite it using EAFP with `try/except`.
   *Focus: pick the narrowest exception type, don't catch broadly.*

2. **PEP 8 cleanup** — Take a deliberately badly-styled snippet (mixed naming conventions, inconsistent spacing, overly long lines) and reformat it by hand first, then verify with a formatter.
   *Hint: notice which fixes a formatter can't make for you (naming, structure).*

3. **Duck-typed function** — Write a function that accepts "anything iterable" and sums its items, and demonstrate it working on a list, a generator, and a custom class implementing `__iter__`.
   *Focus: don't type-check with `isinstance`; rely on the protocol.*

4. **Infinite generator with a stop condition** — Write an infinite generator (e.g., Fibonacci numbers) and consume only the first N values using `itertools.islice`.
   *Hint: never call `list()` directly on an infinite generator.*

5. **`yield from` delegation** — Write a generator that flattens an arbitrarily nested list structure using `yield from`.
   *Focus: recursive generator delegation, not manual recursion with an accumulator list.*

6. **Timing decorator** — Write a `@timed` decorator using `functools.wraps` that prints how long the wrapped function took; prove `wraps` is working by checking `__name__` and `__doc__` are preserved.
   *Focus: without `wraps`, introspection breaks — demonstrate the difference.*

7. **Retry decorator with parameters** — Write a `@retry(times=3, exceptions=(ValueError,))` decorator that retries a flaky function and re-raises after exhausting attempts.
   *Hint: decorator factories need one extra layer of nested functions.*

8. **Context manager two ways** — Implement the same "temporarily change the working directory" context manager once as a class (`__enter__`/`__exit__`) and once using `@contextlib.contextmanager`.
   *Focus: notice how much shorter the generator-based version is, and where cleanup code lives in each.*

9. **Validated property** — Build a class with a `@property`/`@x.setter` pair that rejects invalid values (e.g., a `Temperature` class that raises `ValueError` for a value below absolute zero).
   *Focus: property setters run your validation transparently on plain attribute assignment.*

10. **Dataclass vs NamedTuple vs TypedDict** — Model the same "Point" concept three ways (`@dataclass`, `NamedTuple`, `TypedDict`) and write a short comment under each explaining a scenario where that specific choice is the best fit.
    *Focus: mutability, hashability, and "is it really just a dict" are the deciding factors.*

11. **Protocol vs ABC** — Define the same interface twice: once as an `abc.ABC` with `@abstractmethod`, once as a `typing.Protocol`; write one class that "satisfies" the Protocol without inheriting from it, and show that the ABC version requires explicit inheritance.
    *Focus: this is the "You'll Know You're Ready" checkpoint for this chapter — do this one carefully.*

12. **Type-hint a small module** — Take an untyped 15–20 line module (or your Chapter 0 `task_tracker.py` snippet) and add full type hints (`Optional`/`|None`, `Union`/`|`, and a `Protocol` if applicable), then run `mypy` or `pyright` against it and fix every reported issue.
    *Hint: this previews Chapter 4's tooling — install mypy or pyright standalone just for this exercise.*
