# TaskFlow — Chapter 0 Increment

TaskFlow is the continuous project for this entire syllabus: a task/todo tracker that starts as a single script and evolves, chapter by chapter, into a production-grade, deployed system. This is where it begins.

## Where TaskFlow starts

There is no prior chapter. You are creating `task_tracker.py` from scratch as a **single flat script**, using only core Python (the standard library) — no third-party packages, no package structure, no tests yet. That all comes later. The point of this chapter's increment is to build something that *works*, while deliberately exercising the foundational syntax and stdlib concepts from this chapter's README.

## What to add this chapter

Build `task_tracker.py` as a command-line task tracker with the following:

1. **Task storage** — an in-memory list of task dicts or lightweight objects, each with at least: `id`, `title`, `done` (bool), `created_at` (datetime), `priority` (e.g., "low"/"medium"/"high").
2. **A `Task` class with at least 4 dunder methods** — e.g., `__init__`, `__repr__` (so printing a task is readable), `__eq__` (two tasks are equal if their `id` matches), and `__len__` (e.g., return the length of the title) or `__lt__` (so tasks can be sorted by priority/date).
3. **Core operations as functions**: `add_task`, `complete_task`, `remove_task`, `list_tasks` (support filtering by `done`/priority using a comprehension), `find_task` (using `for-else` to report "not found" cleanly).
4. **A deliberately planted mutable-default-argument bug**, then fixed — for example a function like `def add_task(title, tags=[])` that reuses a shared list across calls; demonstrate the bug in a comment or a small `if __name__ == "__demo__":` block, then correct it with the `None` sentinel idiom. Leave a code comment explaining what the bug was and why the fix works — this becomes a teaching artifact within the project itself.
5. **Persistence using `pathlib`** — save/load tasks to/from a JSON file (`tasks.json`) using `pathlib.Path` exclusively (no `os.path`). Use `Path.exists()`, `Path.read_text()` / `Path.write_text()` (or `open()` via a `Path` object used as a context manager).
6. **Command handling with `match-case`** — parse simple CLI commands (`"add Buy milk"`, `"done 3"`, `"list"`, `"remove 2"`, `"quit"`) using structural pattern matching instead of a long `if/elif` chain.
7. **Custom exceptions** — define at least one custom exception (e.g., `TaskNotFoundError`) and raise it with proper exception chaining (`raise TaskNotFoundError(...) from e`) where a lower-level lookup fails.
8. **Basic error handling loop** — the CLI's main loop should use `try/except/else/finally` around each command so a bad command doesn't crash the whole session.
9. **A little `itertools`/`collections` usage** — e.g., `collections.Counter` to summarize tasks by priority, or `itertools.groupby` to group tasks by completion status for the `list` command's output.

## Concrete tasks checklist

- [ ] `task_tracker.py` created as a single flat script (no subfolders, no `pyproject.toml` yet).
- [ ] `Task` class with 4+ dunder methods implemented and demonstrated.
- [ ] CRUD-style operations for tasks (add/complete/remove/list/find) implemented as plain functions.
- [ ] Mutable-default bug planted, demonstrated, then fixed with a comment explaining the trap.
- [ ] JSON persistence implemented using only `pathlib` (no `os.path` anywhere in the file).
- [ ] CLI command loop implemented using `match-case`.
- [ ] At least one custom exception class, raised with `raise ... from e` chaining somewhere.
- [ ] Main loop wrapped in `try/except/else/finally` so the app doesn't crash on bad input.

## Definition of done

- [ ] Running `python task_tracker.py` launches an interactive loop that accepts `add`, `list`, `done`, `remove`, and `quit` commands.
- [ ] Tasks persist across runs via `tasks.json` (created, read, and written using `pathlib`).
- [ ] The mutable-default bug and its fix are both visible in the code with an explanatory comment — this is intentionally left in as a teaching artifact, not deleted.
- [ ] The script runs top-to-bottom with no unhandled exceptions when given a mix of valid and invalid commands.
- [ ] All code lives in exactly one file: `task_tracker.py`. No packaging, no external dependencies, no tests — those arrive in later chapters.
