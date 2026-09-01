# Chapter 3 — Practice Problems

Standalone exercises, ordered easy to hard. No solutions provided — write the tests (and, where noted, the code) yourself.

1. **Test a pure function.** Write `pytest` unit tests for a `def clamp(value, low, high)` function, covering the in-range, below-low, and above-high cases.
   *Hint: three simple example-based tests are enough here — this is about getting the pytest mechanics right, not cleverness.*

2. **Fixture basics.** Write a fixture that returns a fresh empty `list`-backed stack class instance, and use it in three different tests (`push`, `pop`, `pop on empty raises`).
   *Focus: function-scoped fixtures re-run per test — prove it by mutating the stack in one test and showing the next test still starts empty.*

3. **Parametrize a validator.** Write `is_valid_email(s)` (simple regex is fine) and a single parametrized test covering at least 6 valid/invalid cases.
   *Hint: put both the input and expected boolean in each parametrize tuple; give each case an `id=` for readable output.*

4. **Exception testing.** Write a `Stack.pop()` that raises `IndexError` on empty, and a test using `pytest.raises` that also checks the exception message.
   *Focus: use `pytest.raises(IndexError, match=...)` rather than only checking the exception type.*

5. **conftest.py sharing.** Split a fixture used by two different test files into a shared `conftest.py` and confirm both files pass without importing it.
   *Hint: conftest.py must live in a common parent directory of both test files; no import statement is needed or possible.*

6. **Mocking time.** Write a function `is_task_overdue(due_date)` that compares against "now," then write a test that mocks/patches "now" to a fixed value so the test isn't date-dependent.
   *Focus: patch the specific function/attribute that reads the clock, not the whole `datetime` module if you can avoid it.*

7. **Mocking an external call.** Write a function that fetches a URL via `requests.get` and returns parsed JSON; write a test using `unittest.mock.patch` that never touches the network.
   *Hint: patch where the name is *used* (e.g., `mymodule.requests.get`), not where it's defined.*

8. **Boundary values.** Write and test a `paginate(items, page, page_size)` function, with explicit tests for page 0/1, the last partial page, and an empty list.
   *Focus: enumerate every boundary before writing any test — empty input, single item, exact multiple of page_size, one more than a multiple.*

9. **TDD from scratch.** Using red-green-refactor, build a `RomanNumeralConverter.to_roman(n)` function one failing test at a time (start with `1 -> "I"`, then `4 -> "IV"`, etc.).
   *Hint: resist writing the general algorithm on test one — let each new failing test force the next increment of logic.*

10. **Coverage gaps.** Given a `discount(price, code)` function with 4 branches, write tests that hit 100% line coverage but deliberately leave one branch's *correctness* unverified (a wrong return value that still executes the line); then explain in a comment why coverage passed but the bug survived.
    *Focus: this is about demonstrating coverage's blind spot, not writing a "good" test — you're proving a point.*

11. **Property-based testing.** Using `hypothesis`, write a property test for a `reverse_words(s)` function asserting that reversing twice returns the original string, for arbitrary generated strings.
    *Hint: use `st.text()`; consider what happens with strings containing multiple spaces or no spaces at all.*

12. **Async test.** Write an `async def fetch_all(urls)` that gathers results from an async `fetch_one` mock, and test it with `pytest-asyncio`, mocking `fetch_one` with `AsyncMock`.
    *Focus: use `AsyncMock`, not `Mock`, for the coroutine function, and mark the test `@pytest.mark.asyncio`.*
