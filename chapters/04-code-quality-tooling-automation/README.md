# Chapter 4 — Code Quality, Tooling & Automation

Goal: let tools catch what humans shouldn't have to.

---

## Linters & Formatters: ruff, black, isort (or ruff doing all three now), flake8 legacy context

A **linter** analyzes source code without running it (static analysis) to flag style violations, likely bugs, and anti-patterns — unused imports, undefined names, shadowed builtins, overly complex functions. A **formatter** rewrites code into a single canonical style (line length, quote style, spacing) so that style debates disappear from code review entirely — code is either "formatted" or it isn't, with no room for opinion.

**Historically:** `flake8` (linting) + `black` (formatting) + `isort` (import sorting) were run together as three separate tools with three separate configs. This worked but was slow (three subprocess invocations) and occasionally the tools disagreed with each other.

**`ruff`** is a single, extremely fast (Rust-implemented) tool that now covers linting, formatting (a black-compatible formatter), and import sorting (isort-compatible) in one binary and one config block in `pyproject.toml`. It reimplements the rules of dozens of legacy plugins (`flake8-bugbear`, `pyupgrade`, `pycodestyle`, `pyflakes`, and more) under one roof, so most new Python projects in 2025+ standardize on ruff alone rather than assembling flake8 + black + isort + a dozen flake8 plugins.

```toml
[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP", "SIM"]  # pycodestyle, pyflakes, isort, bugbear, pyupgrade, simplify

[tool.ruff.format]
quote-style = "double"
```

`ruff check .` lints; `ruff check --fix .` auto-fixes what it safely can; `ruff format .` formats. **Common pitfall:** enabling every available rule set on day one on an existing codebase produces thousands of violations and demoralizes the team — start with a conservative rule set (`E`, `F`, `I`) and expand gradually, using `# noqa` / per-file ignores sparingly and deliberately rather than blanket-disabling.

---

## Static Type Checking: mypy / pyright, Gradual Typing Strategy for Existing Codebases

Python's type hints (from Chapter 1) are not enforced at runtime — a static type checker like **mypy** or **pyright** reads your annotations and flags mismatches *before* the code ever runs, catching a whole category of bugs (passing `None` where a non-optional value was expected, calling a method that doesn't exist on a given type, mismatched return types) at "compile" time instead of in production.

`mypy` is the original, most configurable checker; `pyright` (from Microsoft, powers Pylance in VS Code) is generally faster and has excellent editor integration. Both are valid choices; many teams use pyright in the editor for instant feedback and mypy in CI for a stricter gate, or just pick one and standardize.

**Gradual typing** is the strategy for introducing types into an existing, untyped codebase without a big-bang rewrite: start with `mypy --strict` disabled and a permissive baseline, type the most critical/public modules first (public APIs, core domain models), and use per-module strictness overrides to ratchet up requirements module by module as they get typed, rather than requiring 100% typed code on day one.

```toml
[tool.mypy]
python_version = "3.11"
warn_unused_ignores = true
disallow_untyped_defs = false  # start permissive

[[tool.mypy.overrides]]
module = "taskflow.domain.*"
disallow_untyped_defs = true  # ratchet up on the core domain first
```

**Common pitfalls:** sprinkling `# type: ignore` everywhere to silence errors without understanding them (defeats the purpose — use `warn_unused_ignores` to catch stale ignores); over-using `Any`, which silently opts entire subtrees out of checking; and forgetting that type checkers only see what's annotated — an untyped third-party library without stubs will be treated as `Any` unless you install a `types-*` stub package or write a minimal stub yourself.

---

## Pre-commit Hooks (the pre-commit framework)

A **pre-commit hook** is a script that Git runs automatically before a commit is finalized; if the script exits non-zero, the commit is aborted. The **`pre-commit`** framework (a Python tool, `pip install pre-commit`) manages a declarative list of such hooks — linters, formatters, type checkers, secret scanners — configured in a `.pre-commit-config.yaml` file, and installs them into `.git/hooks/pre-commit` via `pre-commit install`.

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.9
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.11.2
    hooks:
      - id: mypy
        additional_dependencies: [types-requests]
```

Once installed, `git commit` automatically runs ruff, mypy, and any other configured hook against the staged files, and blocks the commit if any hook fails — this is what turns "please remember to run the linter" into "it is physically impossible to commit unlinted code." Run `pre-commit run --all-files` to check the whole repo (useful in CI as a second safety net, since local hooks can be bypassed with `git commit --no-verify`). **Common pitfall:** hooks that are slow (a full mypy run on a large codebase) frustrate developers into using `--no-verify` habitually — keep pre-commit hooks fast by scoping them to changed files where possible, and push slower/exhaustive checks to CI instead.

---

## Git Hygiene: Meaningful Commits, .gitignore, Branching Strategies, Conventional Commits

**Meaningful commits** are small, focused, and describe *why* a change was made, not just *what* changed (the diff already shows what). A commit that bundles an unrelated formatting pass, a bug fix, and a new feature together is hard to review, hard to revert, and hard to bisect later with `git bisect`.

**`.gitignore`** excludes files that shouldn't be tracked — build artifacts, virtual environments, `__pycache__/`, `.env` files with secrets, IDE config. Committing a `.env` file with real credentials is one of the most common and damaging mistakes new teams make; a good `.gitignore` from day one (e.g., via `gitignore.io` or GitHub's Python template) prevents this class of accident.

**Branching strategies:** **trunk-based development** keeps everyone merging small, frequent changes directly into a single long-lived `main` branch (often behind feature flags for incomplete work), favoring continuous integration and small PRs. **GitFlow** uses longer-lived `develop`, `release`, and `feature/*` branches with a more ceremonial merge process — historically popular, but often considered heavyweight for teams doing continuous deployment today. Most modern, CI/CD-driven teams lean trunk-based with short-lived feature branches and frequent merges.

**Conventional Commits** is a lightweight convention for commit messages (`feat: add due-date filtering`, `fix: correct off-by-one in pagination`, `chore: bump ruff version`) that makes history scannable and enables tooling — automatic changelog generation and semantic-version bumping based on the commit type (a `feat:` bumps minor, a `fix:` bumps patch, a `BREAKING CHANGE:` footer bumps major).

---

## Code Review Etiquette: What to Look For, How to Give/Receive Feedback

Good code review checks, roughly in this order: **correctness** (does it do what it claims, are edge cases handled), **tests** (is the new behavior actually tested), **design** (is this the right abstraction, does it fit the codebase's architecture), and only then **style/nits** (which automated tooling should mostly have already caught, making human review time more valuable).

**Giving feedback:** phrase comments as questions or suggestions rather than commands ("What happens here if the list is empty?" rather than "This is wrong"), distinguish blocking issues from optional nits (many teams prefix nits with "nit:"), and comment on the code, never the person. Praise good decisions too, not just problems — review isn't only a bug hunt.

**Receiving feedback:** treat it as a review of the code, not of you; ask for clarification rather than getting defensive when a comment is unclear; and it's fine to push back with reasoning ("I chose this because X") — a good review is a conversation, not a one-way verdict. **Common pitfall:** reviews that only nitpick formatting/naming while missing a real correctness bug — this is exactly the failure mode automated linting/formatting is meant to eliminate, freeing human attention for the things only humans can judge.

---

## Docstrings & Documentation Standards (Google Style, NumPy Style, Sphinx-Compatible)

A docstring documents *why* and *how to use* a function/class/module, not a restatement of the code. Three conventions dominate:

- **Google style** — readable, section-header based (`Args:`, `Returns:`, `Raises:`), popular for its readability in plain source.
- **NumPy style** — similar intent, uses underlined section headers, common in the scientific Python ecosystem (NumPy, SciPy, pandas).
- **Sphinx/reST style** — uses `:param:`/`:returns:`/`:raises:` field lists, the traditional format for Sphinx's `autodoc`.

```python
def create_task(title: str, priority: int = 1) -> Task:
    """Create a new task.

    Args:
        title: Human-readable task title. Must be non-empty.
        priority: Priority from 1 (low) to 5 (high). Defaults to 1.

    Returns:
        The newly created Task, already persisted to the store.

    Raises:
        ValueError: If title is empty or priority is out of range.
    """
```

Both Google and NumPy styles are parseable by Sphinx via the `napoleon` extension, so you get auto-generated docs regardless of which readable style you pick — reserve raw reST field-list style for when you need Sphinx-specific directives inline. **Common pitfall:** docstrings that just repeat the signature ("title: the title") without adding information (constraints, units, side effects) — a docstring should tell the reader something the type hints alone don't.

---

## Makefiles & Task Runners (invoke, just, nox, tox) for Standardizing Commands

Every project accumulates a set of routine commands — run tests, lint, format, build docs, run the app locally. A **task runner** gives these a single, memorable, project-agnostic interface (`make test`, `just test`, `nox -s tests`) instead of every developer needing to remember (or `grep` the README for) the exact `pytest --cov=src --cov-report=html -x` invocation.

- **Make/Makefile** — ubiquitous, no extra install (on Unix; less native on Windows), but Make's syntax (tabs, `.PHONY`, real dependency tracking) is a common source of confusion when used purely as a command runner rather than a build system.
- **`just`** — a modern Make-alternative built specifically to be a friendly command runner, with saner syntax and no tab/dependency-tracking baggage.
- **`nox`** / **`tox`** — Python-native task runners specifically strong at running the same test suite across multiple Python versions/environments (e.g., "test on 3.10, 3.11, 3.12 in isolated virtualenvs") — heavier weight than `just`/`make` but purpose-built for the "test matrix" problem.

```
# justfile
test:
    pytest --cov=src

lint:
    ruff check .
    mypy src

check: lint test
```

The point of any of these tools is the same: reduce the activation energy for running quality gates so that "did you run the tests" stops being a question anyone has to ask — `just check` (or equivalent) becomes muscle memory, and CI runs the identical command so local and CI results never diverge.
