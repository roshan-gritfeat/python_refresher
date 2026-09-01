# Chapter 4 — Practice Problems

Standalone exercises, ordered easy to hard. No solutions provided.

1. **Run ruff on messy code.** Take a deliberately sloppy 30-line script (unused imports, inconsistent quotes, unsorted imports) and run `ruff check` then `ruff format` on it; note what each stage fixed.
   *Focus: distinguish which fixes came from linting (`check --fix`) vs formatting (`format`).*

2. **Configure ruff rule sets.** In a `pyproject.toml`, configure `[tool.ruff.lint]` to enable pyflakes, pycodestyle, isort, and bugbear, and explain in a comment what each selector letter covers.
   *Hint: the rule prefixes are `F`, `E`/`W`, `I`, `B`.*

3. **Fix a mypy error.** Given a function with a type annotation mismatch (e.g., a function typed to return `str` but with a branch returning `None`), run mypy, read the error, and fix it properly (not with `# type: ignore`).
   *Focus: decide whether the fix is `Optional[str]` or removing the `None` branch — think about which is actually correct.*

4. **Gradual typing override.** In a `pyproject.toml` mypy config, set a permissive global default and a strict per-module override for one package.
   *Hint: use `[[tool.mypy.overrides]]` with a `module` glob pattern.*

5. **Write a pre-commit config from scratch.** Create a `.pre-commit-config.yaml` with ruff and a trailing-whitespace/end-of-file-fixer hook, install it, and confirm a bad commit is blocked.
   *Focus: actually run `pre-commit install` and attempt a real `git commit` to see the block happen.*

6. **Break and fix a hook.** Intentionally commit a file with an unused import with pre-commit installed; observe the block, fix the import, and commit successfully.
   *Hint: `git commit --no-verify` would skip this — don't use it; the point is seeing the hook actually stop you.*

7. **Rewrite commit history messages.** Given a list of 5 vague commit messages ("fix stuff", "wip", "update"), rewrite them as Conventional Commits messages that reflect the (given) actual diffs.
   *Focus: pick the right type prefix (`feat`, `fix`, `refactor`, `chore`, `docs`) for each.*

8. **Write Google-style docstrings.** Take 3 undocumented functions of varying complexity (including one that raises an exception) and write complete Google-style docstrings for each.
   *Hint: include `Args`, `Returns`, and `Raises` sections; add information not already obvious from the signature.*

9. **Generate docs from docstrings.** Set up a minimal `mkdocs` (or Sphinx) site that pulls in the docstrings from problem 8 via autodoc/mkdocstrings and builds without warnings.
   *Focus: get the build to run clean — an autodoc warning about a missing module is a very common first-time snag.*

10. **Write a justfile or Makefile.** Create a task runner file with `test`, `lint`, `format`, and a combined `check` target that runs lint+test, for a small existing project.
    *Hint: make `check` simply depend on/call the other two targets rather than duplicating commands.*

11. **Self code-review.** Pick a piece of code you wrote more than a week ago (not from this course) and write a structured review of it as if reviewing a stranger's PR — list at least 5 issues across correctness, tests, and design, not just style.
    *Focus: force yourself past style nits — find at least one design-level or correctness-level issue.*

12. **Full gate simulation.** Combine problems 5, 3, and 10: set up a project where `just check` runs ruff + mypy + pytest, and pre-commit blocks a commit that fails any one of them; demonstrate all three failure modes independently.
    *Hint: test each gate in isolation first (break only lint, only types, only tests) before testing them together.*
