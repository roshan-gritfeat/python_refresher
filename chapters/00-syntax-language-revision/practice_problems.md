# Chapter 0 — Practice Problems

Standalone exercises to drill the concepts in this chapter's README. Ordered easy to hard. No solutions provided — figure it out, then verify by running it.

1. **Identity vs equality sweep** — Write a script that creates two lists with identical contents and compares them with both `==` and `is`; print and explain each result.
   *Focus: understand that `==` compares value, `is` compares identity.*

2. **Interning probe** — Create small integers and large integers, compare with `is`, and observe where CPython's caching stops applying.
   *Hint: try values around -5 to 256, then try 257 and above, both inline and via separate function calls.*

3. **`for-else` search** — Write a function that searches a list for a target value using `for-else` instead of a manual "found" boolean flag.
   *Focus: the `else` clause runs only if `break` was never hit.*

4. **Walrus in a loop** — Rewrite a "read chunks from a list/file until empty" loop to use the walrus operator `:=` in the `while` condition.
   *Hint: avoid computing the same expression twice.*

5. **Pattern matching a command parser** — Use `match-case` to parse simple text commands like `"move north"`, `"pick up sword"`, `"quit"` into different actions, including a wildcard `_` case.
   *Focus: use capture patterns and `*rest` to grab variable-length parts.*

6. **The mutable default trap, on purpose** — Write a function with a mutable default argument, call it multiple times to demonstrate the bug, then fix it with the `None` sentinel idiom.
   *Hint: this is explicitly one of the "Do This" tasks — do it as a clean standalone exercise first.*

7. **Positional-only / keyword-only API** — Design a function signature using `/` and `*` so that some parameters must be positional and others must be keyword-only; write calls that succeed and calls that intentionally raise `TypeError`.
   *Focus: understand why library authors do this for API stability.*

8. **Closures that count** — Write a `make_counter()` function returning an `increment()` closure that remembers and increases a private counter across calls, without using a class or global variable.
   *Hint: you'll need `nonlocal`.*

9. **The loop-variable closure bug** — Reproduce the classic bug where a list of lambdas created in a loop all reference the same final loop variable, then fix it two different ways.
   *Focus: default-argument capture vs `functools.partial`.*

10. **Comprehension vs loop refactor** — Take a nested `for` loop with an `if` filter and rewrite it as a comprehension; then write a case where a comprehension would be *too* clever and a plain loop is clearer — justify your choice in a comment.
    *Focus: know when to stop using comprehensions.*

11. **Exception chaining** — Write code that catches a low-level exception (e.g., `KeyError`) and re-raises it as a custom, domain-specific exception using `raise ... from e`; print the traceback and identify the chained cause.
    *Hint: define at least one custom exception class.*

12. **Four-dunder class** — Design a small `Money` or `Vector2D` class implementing at least four dunder methods (`__init__`, `__repr__`, `__eq__`, `__add__` or `__len__`), and write assertions proving each behaves correctly, including an edge case (e.g., comparing to a non-Money object).
    *Focus: dunder methods should behave predictably even with unexpected input types.*
