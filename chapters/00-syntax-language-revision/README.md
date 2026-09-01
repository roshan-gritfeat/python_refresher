# Chapter 0 — Syntax & Language Revision (Foundations Audit)

**Goal:** eliminate blind spots in basic Python that quietly cause bugs later. This chapter assumes you already "know" Python — it exists to surface the gaps that experienced-but-self-taught developers often carry without realizing it.

---

## Data Types Deep Dive

### Mutability: `list`/`dict`/`set` vs `tuple`/`frozenset`/`str`

Python objects are either **mutable** (their internal state can change after creation — `list`, `dict`, `set`, most user-defined classes) or **immutable** (once created, their value can never change — `int`, `float`, `str`, `tuple`, `frozenset`, `bytes`). This distinction matters far more than it looks:

- Immutable objects are safe to share freely — hand a `str` to ten functions and none of them can corrupt it for the others.
- Mutable objects are only safe to share when everyone agrees on who owns the write access. Passing a `list` into a function and having that function `.append()` to it silently changes the caller's data too — there is no copy-on-write in Python for these types.
- Immutable objects can be dictionary keys and set members (they must be hashable); mutable ones generally cannot (`TypeError: unhashable type: 'list'`).

A common pitfall: `tuple` is immutable, but only shallowly. `t = ([1,2], 3)` — you cannot reassign `t[0]`, but you *can* mutate the list living inside it: `t[0].append(4)`. "Immutable" describes the container's own identity/structure, not necessarily everything reachable from it.

```python
a = [1, 2, 3]
b = a          # b is the SAME list object, not a copy
b.append(4)
print(a)       # [1, 2, 3, 4]  <- surprise for beginners

c = a.copy()   # or list(a), or a[:]
c.append(5)
print(a)       # unaffected
```

### Identity vs Equality: `is` vs `==`

`==` asks "do these objects have the same *value*?" (it calls `__eq__`). `is` asks "are these the *exact same object in memory*?" (it compares `id()`). They are not interchangeable:

```python
a = [1, 2, 3]
b = [1, 2, 3]
a == b   # True  -> same value
a is b   # False -> different objects in memory
```

The classic misuse is `if x == None`, which works but is non-idiomatic and can be fooled by a class overriding `__eq__`. The convention is `if x is None`, because there is only ever one `None` object and identity comparison is both faster and unambiguous. The same applies to `True`/`False` in some style guides, though `if x:` is usually preferred over `if x is True:`.

### Interning

CPython caches ("interns") certain small, immutable objects so that multiple references to "the same value" actually point to the same object — small integers (-5 to 256) and many string literals that look like identifiers. This is why:

```python
a = 256
b = 256
a is b   # True (small-int cache)

a = 257
b = 257
a is b   # True in a REPL "single statement" sometimes, but NOT guaranteed across separate compiled units
```

**Pitfall:** never rely on interning for correctness. `a is b` "working" for small integers or short strings is a CPython implementation detail, not a language guarantee. Always use `==` for value comparison; reserve `is` for singleton checks (`None`, `True`, `False`, sentinel objects you defined yourself).

---

## Control Flow

### `for-else` and `while-else`

The `else` clause on a loop runs when the loop completes **without hitting a `break`**. It's one of Python's most under-used and most misunderstood features because "else" reads like "otherwise," but it really means "the loop wasn't interrupted."

```python
def find_first_even(nums):
    for n in nums:
        if n % 2 == 0:
            print("found", n)
            break
    else:
        print("no even number found")
```

This eliminates a manual "found" flag variable that beginners often reach for. Common pitfall: forgetting that `else` runs even if the loop body executed zero times (e.g., iterating an empty list) — in that case there was no `break`, so `else` still fires.

### Walrus Operator `:=`

Introduced in Python 3.8, `:=` lets you assign a value to a variable *as part of an expression*, most commonly to avoid computing something twice:

```python
# Before
data = fetch()
if data:
    process(data)

# With walrus, inline in a while/if condition
while (chunk := file.read(8192)):
    process(chunk)

if (n := len(items)) > 10:
    print(f"too many items: {n}")
```

Pitfall: overusing it purely to "look clever" hurts readability. It shines specifically when you'd otherwise duplicate an expression or need an extra line just to hold a temporary.

### `match-case` (Structural Pattern Matching)

Added in 3.10, `match` is not a simple `switch` — it does **structural pattern matching**, meaning it can destructure the shape of the data as part of matching:

```python
match command.split():
    case ["go", direction]:
        move(direction)
    case ["take", *items]:
        take(items)
    case ["look"]:
        describe_room()
    case _:
        print("unknown command")
```

It can match against class shapes too (`case Point(x=0, y=0):`), guards (`case [x, y] if x == y:`), and literal values. Pitfall: a bare name in a pattern (like `direction` above) is treated as a **capture variable**, not a value to compare against — to match a variable's existing value, use a guard or prefix with a dot (`case SOME_CONSTANT:` only works if it's a dotted/qualified name or enum member, not a plain local variable).

---

## Functions

### Default Arguments and the Mutable Default Trap

Default argument values are evaluated **once**, when the `def` statement executes — not on every call. If the default is mutable, every call that doesn't supply that argument shares the *same* object.

```python
def add_item(item, bucket=[]):   # DANGER
    bucket.append(item)
    return bucket

add_item("a")   # ['a']
add_item("b")   # ['a', 'b']  <- unexpected! same list reused
```

The fix is the idiomatic `None` sentinel pattern:

```python
def add_item(item, bucket=None):
    if bucket is None:
        bucket = []
    bucket.append(item)
    return bucket
```

This bug is infamous precisely because it doesn't show up until the function is called more than once without the optional argument — it can hide in production for a long time.

### `*args`, `**kwargs`

`*args` collects extra positional arguments into a tuple; `**kwargs` collects extra keyword arguments into a dict. They enable flexible, forwarding-friendly APIs (decorators, wrappers) and are essential vocabulary for reading library code:

```python
def logged(func):
    def wrapper(*args, **kwargs):
        print(f"calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper
```

### Keyword-Only and Positional-Only Parameters (`*`, `/`)

A bare `*` in a parameter list forces everything after it to be passed by keyword only. A bare `/` forces everything before it to be positional only.

```python
def connect(host, port, /, *, timeout=5, retries=3):
    ...

connect("localhost", 8080, timeout=10)   # OK
connect(host="localhost", port=8080)      # TypeError: host/port are positional-only
connect("localhost", 8080, 10)            # TypeError: timeout/retries are keyword-only
```

This matters for API design: positional-only parameters let you rename internal parameter names later without breaking callers; keyword-only parameters force clarity at call sites for options that are easy to confuse (e.g., booleans, similarly-typed values).

### Closures and First-Class Functions

Functions are objects — you can assign them to variables, store them in lists, pass them as arguments, and return them from other functions. A **closure** is a function that "remembers" variables from the scope in which it was defined, even after that outer scope has finished executing.

```python
def make_multiplier(factor):
    def multiplier(x):
        return x * factor   # `factor` is captured from the enclosing scope
    return multiplier

double = make_multiplier(2)
double(5)   # 10
```

Pitfall: closures capture variables **by reference**, not by value at definition time. The classic loop bug:

```python
funcs = [lambda: i for i in range(3)]
[f() for f in funcs]   # [2, 2, 2], not [0, 1, 2]
```

All three lambdas share the same `i`, whose final value is `2` by the time they're called. Fix with a default-argument capture: `lambda i=i: i`.

---

## Comprehensions

List, dict, set, and generator comprehensions are concise ways to build collections from iterables:

```python
squares = [x**2 for x in range(10)]                # list
lookup = {x: x**2 for x in range(10)}               # dict
uniq = {x % 3 for x in range(10)}                   # set
lazy = (x**2 for x in range(10))                    # generator (lazy, no memory blowup)
```

Nested comprehensions flatten nested loops in a single expression: `[x for row in matrix for x in row]`. The rule of thumb for **when not to use them**: if you need more than one `if`/`for` clause, or the expression needs a comment to explain what it does, write a regular loop instead. Comprehensions optimize for *reading a transformation at a glance* — once they require squinting, they've defeated their own purpose.

---

## String Formatting

- **f-strings** (`f"{name} is {age} years old"`) are the modern default: fast, readable, support inline expressions and format specs (`f"{value:.2f}"`).
- **`.format()`** is the pre-f-string mechanism, still seen in older code and useful when the template string itself is data (e.g., loaded from a config file) rather than a literal.
- **Byte strings** (`b"hello"`) represent raw bytes, not text — necessary for binary I/O, network protocols, and hashing.
- **Encoding/decoding**: text (`str`) becomes bytes via `.encode(encoding)`, and bytes become text via `.decode(encoding)`. UTF-8 is the near-universal default. Mixing `str` and `bytes` without an explicit encode/decode step is a frequent source of `TypeError` and mojibake bugs.

---

## Scope & Namespaces

### LEGB Rule

When Python resolves a name, it searches in order: **L**ocal → **E**nclosing → **G**lobal → **B**uilt-in. Understanding this order explains why a variable defined in an outer function is visible inside a nested function (Enclosing), and why shadowing a builtin name (like `list = [1,2,3]`) silently breaks later code that expects `list` to be the type.

### `global` and `nonlocal`

By default, assigning to a name inside a function creates a *new local* variable, even if a global of the same name exists — reading is transparent, but writing is not. `global` tells Python "this assignment refers to the module-level variable." `nonlocal` does the same for a variable in an *enclosing function* (not global), used in closures that need to mutate captured state.

```python
counter = 0
def increment():
    global counter
    counter += 1
```

---

## Error Handling

### Exception Hierarchy and `try/except/else/finally`

All exceptions derive from `BaseException`, with `Exception` as the practical root for application errors. Catching too broadly (`except Exception:` or worse, bare `except:`) hides bugs; catch the most specific exception you can meaningfully handle.

- `else` on a `try` block runs only if no exception occurred — useful for code that should run only on success but that you don't want wrapped in the `try` (so it doesn't accidentally get its own exceptions swallowed).
- `finally` always runs, exception or not — used for cleanup that must happen regardless.

### Custom Exceptions and Exception Chaining

Define your own exception types by subclassing `Exception` to give callers a precise, catchable vocabulary for your domain's failure modes:

```python
class InsufficientFundsError(Exception):
    pass

try:
    withdraw(amount)
except KeyError as e:
    raise InsufficientFundsError("account not found") from e
```

`raise ... from e` sets `__cause__`, preserving the original traceback context so debugging doesn't lose the root cause — this is exception chaining, and omitting it (or worse, swallowing the original with a bare `raise NewError()`) makes production debugging much harder.

---

## File I/O and Context Managers (`with`)

The `with` statement guarantees cleanup (closing a file, releasing a lock) even if an exception is raised inside the block, by relying on the object's `__enter__`/`__exit__` protocol:

```python
with open("data.txt") as f:
    contents = f.read()
# file is guaranteed closed here, even if .read() raised
```

Pitfall: `f = open(...)` without `with` (or a `try/finally`) risks leaking file handles, especially in long-running processes — errors before an explicit `f.close()` will skip it.

---

## Basic OOP Recap

### Classes, `self`, Inheritance, `super()`

`self` is the conventional name for the instance being operated on — it's passed automatically when you call `instance.method()`. `super()` gives access to the parent class's methods, most commonly inside `__init__` to ensure the parent is properly initialized before subclass-specific setup:

```python
class Animal:
    def __init__(self, name):
        self.name = name

class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)
        self.breed = breed
```

### Dunder Methods

"Dunder" (double underscore) methods let your objects integrate with Python's built-in syntax and protocols:

- `__init__` — constructor/initializer.
- `__repr__` — unambiguous, developer-facing representation (`repr(obj)`, shown in REPL/debugger); should ideally be code that could recreate the object.
- `__eq__` — defines what `==` means for your objects (without it, equality falls back to identity).
- `__len__` — makes `len(obj)` work, and also makes empty instances falsy in boolean contexts.
- Others worth knowing: `__str__` (human-readable, used by `print()`), `__hash__` (needed if instances are used as dict keys/set members and you defined `__eq__`), `__iter__`/`__next__` (iteration protocol), `__lt__`/`__gt__` (ordering).

Pitfall: defining `__eq__` without `__hash__` makes instances unhashable by default (Python sets `__hash__` to `None` automatically in that case) — this can silently break code that used to put instances in a set or dict.

---

## Standard Library Tour

- **`collections`** — `Counter` (frequency counting), `defaultdict` (avoids `KeyError` boilerplate), `namedtuple`/`OrderedDict`, `deque` (O(1) appends/pops from both ends, ideal for queues).
- **`itertools`** — building blocks for iterator algebra: `chain`, `product`, `combinations`, `groupby`, `islice`, infinite iterators like `count` and `cycle`. Essential for writing memory-efficient, composable data pipelines.
- **`functools`** — `reduce`, `lru_cache` (memoization), `partial` (pre-fill arguments), `wraps` (preserve metadata in decorators).
- **`datetime`** — `datetime`, `date`, `timedelta`, timezone-aware vs naive datetimes (a huge source of production bugs — always know which one you're dealing with).
- **`pathlib`** — object-oriented filesystem paths (`Path("dir") / "file.txt"`), replacing most `os.path` string-munging with cleaner, cross-platform-safe code.
- **`re`** — regular expressions for pattern matching in text; know `match` vs `search` vs `findall`, and when a regex is overkill for a job `str.split`/`str.startswith` could do more clearly.

---

## Do This — Hands-On Tasks

1. Write a short script using `pathlib` instead of `os.path` for file operations (listing a directory, joining paths, checking existence, reading a file).
2. Implement a class with at least four dunder methods (e.g., `__init__`, `__repr__`, `__eq__`, `__len__`) and write assertions/prints proving each one works.
3. Intentionally write a function with a mutable default argument, demonstrate the bug with two calls, then fix it using the `None` sentinel pattern.

## You'll Know You're Ready When...

You can explain, from memory and out loud, why `def f(x=[])` is dangerous — and you can read someone else's comprehension-heavy code without having to squint or re-read it three times.
