# TaskFlow — Chapter 1 Increment

## Where TaskFlow stands

After Chapter 0, TaskFlow is a single flat script, `task_tracker.py`, using only core Python: a `Task` class with basic dunder methods, plain functions for add/complete/remove/list/find, `pathlib`-based JSON persistence, a `match-case` command loop, and a fixed mutable-default bug left in as a comment for reference.

## What to add this chapter

This chapter does not add new *features* — it refactors the existing script into idiomatic Python, per the syllabus's "Do This." Specifically:

1. **Replace list-building loops with comprehensions/generators.** Anywhere `task_tracker.py` currently builds a filtered list with a manual `for` loop and `.append()`, rewrite it as a list or generator comprehension (e.g., `list_tasks(done=True)`).
2. **Convert the `Task` class to a `@dataclass`.** Use `field(default_factory=list)` for any mutable fields (like `tags`), which also serves as a live demonstration of *avoiding* the Chapter 0 mutable-default bug at the class level, not just in functions. Keep (or re-add) the custom `__lt__`/`__eq__` behavior needed for sorting/deduping if `@dataclass`'s generated methods aren't sufficient — and write a one-paragraph comment explaining why you did or didn't rely on the auto-generated `__eq__`.
3. **Add a `@timed` decorator** and apply it to the persistence functions (`save_tasks`/`load_tasks`) so every save/load prints its execution time. Must use `functools.wraps`.
4. **Add a `@retry` decorator** (parameterized, e.g. `@retry(times=3)`) and apply it to the file-loading function, so a transient read failure (simulate with a randomly-raised exception or a locked file scenario) is retried before giving up.
5. **Convert file writing/reading into a custom context manager.** Implement a `TaskFileSession` context manager (via a class with `__enter__`/`__exit__`, or `@contextlib.contextmanager`) that opens the task file, yields the loaded task list, and guarantees the (possibly modified) list is saved back on exit — even if an exception occurs mid-session.
6. **Replace EAFP-hostile lookups with EAFP.** Wherever the script currently does `if task_id in task_ids: ...` before acting, replace it with a `try/except TaskNotFoundError` pattern feeding into the existing custom exception from Chapter 0.
7. **Add type hints throughout** — function signatures, the `Task` dataclass fields, and a `Protocol` (e.g., `Persistable` with a `save`/`load` shape) if you introduce any pluggable storage concept. Run `mypy` or `pyright` against the file and fix all reported issues.
8. **Turn the "list tasks" feature into a generator-based pipeline** — e.g., a `iter_tasks(filter_fn)` generator that lazily yields matching tasks, composed with `itertools` (e.g., `itertools.islice` for "show top N tasks").

## Concrete tasks checklist

- [ ] `Task` is now a `@dataclass` with type-annotated fields and `field(default_factory=...)` for mutable defaults.
- [ ] At least one list-building loop replaced with a comprehension, and at least one lazy generator introduced (e.g., `iter_tasks`).
- [ ] `@timed` decorator implemented with `functools.wraps` and applied to at least one function.
- [ ] `@retry` decorator implemented (parameterized) and applied to at least one function.
- [ ] `TaskFileSession` context manager implemented and replacing manual open/save calls.
- [ ] At least one LBYL check replaced with EAFP `try/except`.
- [ ] Full type hints added; `mypy`/`pyright` runs clean (or documented exceptions with `# type: ignore` and a reason).

## Definition of done

- [ ] `task_tracker.py` (or a renamed/split version, still a single file or a couple of files at most — full packaging is Chapter 2) runs identically in behavior to the Chapter 0 version from the user's perspective.
- [ ] Every function above a few lines has a type hint on its signature and return type.
- [ ] Running a static type checker against the file reports zero unresolved errors.
- [ ] The mutable-default bug from Chapter 0 is now demonstrably impossible at the dataclass level (`field(default_factory=list)`), and a comment explains the connection back to the Chapter 0 lesson.
- [ ] A short `CHANGELOG` note (can be a comment block at the top of the file, or a `NOTES.md`) lists what idiom was applied where, so the refactor's intent is traceable.
