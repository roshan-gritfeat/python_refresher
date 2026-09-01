# Chapter 8 — Design Patterns & Software Architecture

Goal: structure code so it survives requirement changes without collapsing.

## SOLID principles, applied specifically to Python

SOLID is five design guidelines aimed at reducing coupling and increasing cohesion. They originated in a statically-typed, class-heavy OOP world (Java/C#), so applying them to Python requires some translation.

- **Single Responsibility Principle (SRP):** a class/module should have one reason to change. In Python, this often means splitting a class that both validates data, talks to a database, and formats output into three collaborating pieces.
- **Open/Closed Principle (OCP):** code should be open for extension, closed for modification. In Python, this is often achieved not through inheritance hierarchies but through composition, duck typing, and passing in behavior as functions/callables — e.g., a `sort(key=...)` style API rather than subclassing a `Sorter` base class for every variant.
- **Liskov Substitution Principle (LSP):** subtypes must be substitutable for their base types without breaking correctness. Python's duck typing makes violations easy to introduce silently (e.g., a subclass that raises `NotImplementedError` for a method the base class promises) — static type checkers like mypy catch some of this if you use `Protocol`/ABCs deliberately.
- **Interface Segregation Principle (ISP):** don't force clients to depend on methods they don't use. Python's `typing.Protocol` is well-suited here — define small, focused protocols instead of one giant interface.
- **Dependency Inversion Principle (DIP):** high-level modules shouldn't depend on low-level modules; both should depend on abstractions. This is the principle underpinning the repository pattern and hexagonal architecture below — your business logic should depend on an abstract `TaskRepository` interface, not directly on SQLAlchemy.

**Common pitfall:** treating SOLID as a checklist to apply everywhere uniformly. Over-applying DIP/OCP to a tiny script produces needless abstraction layers (see "premature abstraction" below). SOLID earns its keep once a codebase has enough size/change velocity that flexibility actually gets exercised.

## Common patterns

- **Factory:** centralizes object construction, especially when the concrete type to build depends on input (`create_repository(driver="postgres")`). In Python, a factory is often just a function, not a class hierarchy — don't over-engineer a "Factory" class when a function returning an object suffices.
- **Strategy:** encapsulates interchangeable algorithms behind a common interface, selected at runtime. Python's first-class functions make this trivial — you often don't need classes at all: `sorted(data, key=strategy_fn)` *is* the strategy pattern.
- **Observer:** lets subscribers react to events on a subject without tight coupling — e.g., "notify all listeners when a task is completed." Python event systems, signal libraries, or even a simple list of callback functions implement this.
- **Singleton:** ensures a class has only one instance. **In Python this is usually an anti-pattern** — module-level state already gives you a natural, simpler "singleton" (a module is only imported once), and singleton classes tend to introduce hidden global state that makes testing harder (you can't easily swap it out per test). Prefer dependency injection over a Singleton class.
- **Adapter:** wraps an incompatible interface so it matches what your code expects — e.g., wrapping a third-party SMS API client behind your own `NotificationSender` interface so you can swap providers later without touching business logic.
- **Decorator (the pattern, not `@syntax`):** wraps an object to add behavior transparently while preserving its interface — e.g., a `CachingRepository` that wraps a real `TaskRepository` and adds caching without either side knowing about the other's internals. Note this is a *design* pattern about object composition, distinct from Python's `@decorator` *syntax* for functions (Chapter 1), though the syntax is often used to implement function-level decoration.
- **Dependency Injection (DI):** supplying a component's dependencies from the outside (constructor args, function params) rather than having it construct or look them up itself. This is what makes swapping a real DB repository for a fake in-memory one in tests possible without touching the class under test.

**Common pitfall:** reaching for a named pattern because you know its name, not because the problem calls for it. A Strategy pattern implemented as a full class hierarchy when a plain function argument would do is over-engineering, not good design.

## Clean/hexagonal architecture

**Hexagonal architecture** (a.k.a. "ports and adapters") organizes code so business logic sits at the center, entirely ignorant of frameworks, databases, or transport protocols. Surrounding it are "ports" (interfaces the core defines, like `TaskRepository`) and "adapters" (concrete implementations, like `SqlAlchemyTaskRepository` or `InMemoryTaskRepository`) that plug into those ports.

```
        API layer (FastAPI routes)         <- knows about HTTP
                |
        Service layer (use cases)           <- orchestrates domain logic
                |
        Domain layer (entities, rules)       <- pure Python, no framework imports
                |
        Repository interface (port)
                |
   -------------------------------
   |                             |
SQLAlchemy adapter        In-memory fake adapter
```

The key rule: **dependencies point inward.** The domain layer never imports FastAPI, SQLAlchemy, or Redis. The API layer depends on the service layer; the service layer depends on repository *interfaces*, not concrete database code.

**Why it matters:** this is what lets you swap SQLite for Postgres, or a real repository for an in-memory fake in tests, without touching business logic — proving decoupling isn't just a theoretical claim, it's something you can literally demonstrate by running the same test suite against two different repository implementations.

**Common pitfalls:**
- Doing this halfway — defining a repository interface but then having service-layer code catch and handle SQLAlchemy-specific exceptions, which leaks a framework detail back into logic that's supposed to be framework-agnostic.
- Over-layering a small project — a five-endpoint CRUD app may not need four strict layers; apply this once the domain logic is complex enough to be worth protecting.

## Domain-driven design basics: entities, value objects, repositories

- **Entities** have identity that persists across changes — a `Task` with an `id` is the same task even after its title changes. Equality is based on identity (the ID), not on all field values matching.
- **Value objects** have no identity — they're defined entirely by their values, and are typically immutable. A `Priority` or `DateRange` is a value object: two `Priority(level=3)` instances are interchangeable.
- **Repositories** provide a collection-like interface (`add`, `get`, `list`, `remove`) over persistence, hiding whether the data actually lives in SQLite, Postgres, or memory. The service/domain layer talks to a `TaskRepository` interface, never to SQL or an ORM session directly.

```python
from typing import Protocol

class TaskRepository(Protocol):
    def add(self, task: Task) -> None: ...
    def get(self, task_id: int) -> Task | None: ...
    def list_for_user(self, user_id: int) -> list[Task]: ...
```

**Common pitfall:** letting ORM model classes double as domain entities. A SQLAlchemy model is a persistence concern (it carries session-attachment behavior, lazy loading, etc.); a domain entity should be a plain object (dataclass) with business rules, converted to/from the ORM model at the repository boundary.

## Anti-patterns to avoid

- **God objects:** a single class/module that knows and does far too much (validates input, talks to the DB, sends emails, formats responses). Symptoms: huge files, many unrelated methods, everyone's changes touch the same file. Fix by extracting responsibilities per SRP.
- **Tight coupling:** modules that can't be understood, tested, or changed independently because they reach directly into each other's internals. Fix with clear interfaces and dependency injection.
- **Premature abstraction:** introducing interfaces, plugin systems, or configurability for flexibility nobody has asked for yet ("we might need to support MongoDB someday"). This adds indirection and cognitive load without corresponding benefit; you can *always* introduce an abstraction later once a second real use case appears — that's what refactoring is for.

## Refactoring techniques and code smells

**Code smells** are surface indicators of a deeper design problem, not bugs themselves: long methods, large classes, duplicated code, long parameter lists, feature envy (a method that uses another class's data more than its own), and shotgun surgery (one conceptual change requires editing many unrelated files).

Common refactoring techniques: **Extract Method/Function** (pull a chunk of a long function into a named one), **Extract Class** (split a god object along its responsibilities), **Replace Conditional with Polymorphism** (turn a large `if/elif` dispatching on type into a Strategy-pattern dispatch), **Introduce Parameter Object** (bundle a long parameter list into a value object).

**Common pitfall:** refactoring without a safety net. Refactoring is supposed to change structure without changing behavior — doing it without a passing test suite (Chapter 3) covering the code first means you can't tell whether you preserved behavior or silently broke it.
