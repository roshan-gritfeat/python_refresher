# Chapter 1 — Pythonic & Idiomatic Code

**Goal:** write code that looks like it belongs in the standard library — code that a senior Python developer reads and immediately trusts, because it uses the language's own idioms instead of fighting them.

---

## PEP 8 and Beyond

PEP 8 is Python's official style guide: 4-space indentation, `snake_case` for functions/variables, `PascalCase` for classes, `UPPER_SNAKE_CASE` for constants, meaningful whitespace around operators, and a soft line-length limit (traditionally 79, commonly relaxed to 88–100 by tools like `black`/`ruff`). But "PEP 8 and beyond" means style is bigger than formatting:

- **Naming conveys intent.** `items` vs `filtered_active_items_by_user` — pick names precise enough that a reader doesn't need to open the function body to know what's inside.
- **Consistency at scale matters more than any individual rule.** A codebase where every file makes its own style choices costs every future reader extra cognitive overhead just to context-switch. This is why teams enforce style with tools (Chapter 4) rather than code review nitpicks — humans shouldn't spend attention on things machines can check instantly.
- Style consistency is also a trust signal: a consistently-styled codebase suggests the same care was likely applied to correctness, not just appearance.

**Pitfall:** treating PEP 8 as a checklist to satisfy a linter rather than as a proxy for readability. A perfectly PEP-8-compliant function can still be unreadable if named or structured poorly.

---

## EAFP vs LBYL

**EAFP** ("Easier to Ask Forgiveness than Permission") means: attempt the operation, and handle the exception if it fails. **LBYL** ("Look Before You Leap") means: check preconditions first, then act. Python culture strongly favors EAFP:

```python
# LBYL
if "key" in my_dict:
    value = my_dict["key"]
else:
    value = default

# EAFP (idiomatic)
try:
    value = my_dict["key"]
except KeyError:
    value = default
```

Why EAFP wins in Python: it avoids **race conditions** (the state could change between your check and your use — e.g., checking a file exists, then opening it, and it's deleted in between), and it's often faster in the common case because no upfront check is paid for. The pitfall is using EAFP so broadly that you swallow real bugs — catching `Exception` around a huge block just to avoid thinking about what could actually fail defeats the purpose. EAFP should target one specific, expected failure mode, not become an excuse for sloppy error handling.

---

## Duck Typing and Protocols

"If it walks like a duck and quacks like a duck, treat it as a duck." Python doesn't require an object to inherit from a specific base class to be usable somewhere — it only needs to support the operations actually used on it. This is **structural typing**: a for-loop works on anything implementing `__iter__`; a `with` statement works on anything implementing `__enter__`/`__exit__`, regardless of its class hierarchy.

```python
class FakeFile:
    def read(self):
        return "fake data"

def process(file_like):
    return file_like.read()   # works on a real file OR FakeFile — no shared base class needed
```

`typing.Protocol` (Chapter's type-hints section) formalizes this for static analysis: you can declare "anything with a `.read()` method satisfies this type" without an explicit inheritance relationship. The pitfall of duck typing is when it fails silently — passing an object that's *almost* compatible (e.g., has `read()` but returns bytes when text was expected) produces confusing downstream errors rather than a clear type error upfront; this is exactly the gap static type checking (`mypy`/`pyright`) helps close.

---

## Iterators & Generators

An **iterator** is any object implementing `__iter__` and `__next__`. A **generator** is the easiest way to create one, using `yield`:

```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1
```

Calling `countdown(3)` doesn't run the function body — it returns a generator object. Code only executes up to the next `yield` each time `next()` is called. This **lazy evaluation** means generators can represent sequences too large (or infinite) to hold in memory:

```python
def naturals():
    n = 1
    while True:
        yield n
        n += 1
```

`yield from` delegates iteration to a sub-generator or any iterable, flattening nested generator logic:

```python
def flatten(nested):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item
```

**Generator expressions** (`(x**2 for x in range(10))`) are the lazy cousin of list comprehensions — use them when you're going to consume the values once, in order, without needing random access or `len()`. `itertools` provides battle-tested recipes (`chain`, `islice`, `groupby`, `tee`) that compose with generators without ever materializing a full list. Pitfall: a generator is **exhausted after one full iteration** — trying to loop over it twice silently yields nothing the second time, which is a frequent source of "why is my data missing" bugs.

---

## Decorators

A decorator is a function that wraps another function (or class) to extend its behavior without modifying its source. The `@decorator` syntax is sugar for `func = decorator(func)`.

```python
import functools
import time

def timed(func):
    @functools.wraps(func)   # preserves func.__name__, __doc__, etc.
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

@timed
def slow_add(a, b):
    return a + b
```

**`functools.wraps` is not optional** in real code — without it, the wrapped function loses its original name, docstring, and signature, which breaks introspection, debugging, and tools like Sphinx.

**Parameterized decorators** need an extra layer of nesting, since the decorator itself must be called with arguments before it receives the function:

```python
def retry(times=3):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exc = None
            for _ in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exc = e
            raise last_exc
        return wrapper
    return decorator

@retry(times=5)
def flaky_call():
    ...
```

**Class decorators** apply the same idea to an entire class (e.g., `@dataclass` itself is a class decorator) — they can inject methods, wrap `__init__`, or register the class somewhere. Pitfall: stacking decorators applies them bottom-up, and forgetting `functools.wraps` compounds across a stack, making debugging nearly impossible.

---

## Context Managers

A context manager guarantees setup/teardown around a block via `__enter__`/`__exit__`:

```python
class Timer:
    def __enter__(self):
        self.start = time.perf_counter()
        return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.elapsed = time.perf_counter() - self.start
        return False   # don't suppress exceptions

with Timer() as t:
    do_work()
print(t.elapsed)
```

Returning `True` from `__exit__` **suppresses** any exception raised inside the `with` block — a subtle, easy-to-misuse power that should be used deliberately and documented.

`contextlib.contextmanager` lets you write the same thing as a generator function, which is often more concise for simple cases:

```python
from contextlib import contextmanager

@contextmanager
def timer():
    start = time.perf_counter()
    try:
        yield
    finally:
        print(f"elapsed: {time.perf_counter() - start:.4f}s")

with timer():
    do_work()
```

Everything before `yield` is `__enter__`; everything after (in the `finally`) is `__exit__`. Pitfall: forgetting the `try/finally` around `yield` means an exception inside the `with` block skips your cleanup code entirely.

---

## Descriptors and Properties

A **property** turns attribute access into a method call, letting you add validation or computed values without changing the class's public interface:

```python
class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        return 3.14159 * self.radius ** 2

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("radius cannot be negative")
        self._radius = value
```

Properties are the Pythonic answer to "getters and setters" — you start with plain attributes, and only add a `@property` later if/when you need validation or computed logic, without breaking every caller who wrote `circle.radius`.

**Descriptors** are the lower-level protocol properties are built on (`__get__`, `__set__`, `__delete__`), used when you need the same managed-attribute behavior reused across many classes/attributes (e.g., ORMs implement fields as descriptors). Most application code uses `@property` directly and only reaches for a custom descriptor when building reusable, framework-like attribute behavior.

---

## `dataclasses` vs Plain Classes vs `NamedTuple` vs `TypedDict`

- **Plain class**: full control, but you hand-write `__init__`, `__repr__`, `__eq__` yourself — verbose for simple data containers.
- **`@dataclass`**: generates `__init__`, `__repr__`, `__eq__` (and optionally ordering, immutability via `frozen=True`) from type-annotated fields. Ideal for mutable, class-like data containers with behavior.

```python
from dataclasses import dataclass, field

@dataclass
class Task:
    title: str
    done: bool = False
    tags: list[str] = field(default_factory=list)   # avoids the mutable-default bug!
```

- **`NamedTuple`**: immutable, tuple-based, lightweight — good for simple, hashable records where you also want tuple-like behavior (unpacking, indexing).
- **`TypedDict`**: describes the *shape* of a plain `dict` for static type checkers, without creating a real class or runtime object — useful for JSON-like data (API payloads, config dicts) where you want type safety but need actual `dict` behavior at runtime.

Choose plain classes when you need custom behavior/invariants beyond data storage; `dataclass` when you want a class with methods but boilerplate-free construction; `NamedTuple` for small immutable value tuples; `TypedDict` for typed dict-shaped data that stays a dict.

---

## Type Hints

The `typing` module lets you annotate expected types without changing runtime behavior (Python remains dynamically typed — hints are checked by external tools, not the interpreter, except where you explicitly validate them e.g. with `pydantic`).

```python
from typing import Optional, Union, Generic, TypeVar, Protocol

def greet(name: str) -> str:
    return f"hello {name}"

def find(id: int) -> Optional[Task]:      # Task or None
    ...

def parse(value: Union[int, str]) -> int:  # int or str in, int out
    ...

T = TypeVar("T")
class Stack(Generic[T]):
    def push(self, item: T) -> None: ...
    def pop(self) -> T: ...

class Sized(Protocol):
    def __len__(self) -> int: ...
```

`Optional[X]` is shorthand for `Union[X, None]`. Modern Python (3.10+) lets you write `X | None` and `int | str` directly instead of importing `Optional`/`Union`. `Protocol` enables **structural** typing checks (matching duck typing above) — a class satisfies `Sized` just by having `__len__`, no inheritance needed. `Generic`/`TypeVar` let you write container/utility code that's type-safe for whatever type it's used with.

**Static type checking philosophy**: type hints are documentation that a machine can verify. They don't make code faster or prevent all bugs, but they catch a large class of mistakes (wrong argument types, `None` handling) before runtime, and they make large codebases navigable (IDE autocomplete, "find usages" reliability). The pragmatic approach for existing untyped codebases is *gradual typing*: add hints incrementally, starting with public APIs and the most bug-prone modules, rather than demanding 100% coverage on day one.

---

## Do This — Hands-On Tasks

1. Refactor a beginner-style script (loops + boolean flags everywhere) into idiomatic Python using generators, comprehensions, and context managers.
2. Write two custom decorators: one that times a function's execution, and one that retries a function on failure (parameterized with a retry count).
3. Convert a plain class into a `@dataclass`, then write a short explanation of a situation where you would *not* do that (e.g., when you need custom `__init__` validation logic beyond simple field defaults, or when identity/mutability semantics matter more than data-holding convenience).

## You'll Know You're Ready When...

You can write a decorator and a context manager from scratch, without looking anything up, and you can clearly explain the difference between `Protocol` and `ABC` (structural vs nominal typing — "satisfies the shape" vs "explicitly inherits from").
