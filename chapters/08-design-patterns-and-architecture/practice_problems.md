# Chapter 8 — Practice Problems

Standalone exercises, independent of TaskFlow. Ordered easy to hard.

1. **Spot the SRP violation** — Given a `ReportGenerator` class that fetches data from a DB, formats it as HTML, and emails it, describe how you'd split it into separate responsibilities.
   *Focus: no code required — identify the reasons-to-change for the class as written.*

2. **Strategy without a class hierarchy** — Refactor a function that branches on a `mode` string (`"asc"`, `"desc"`, `"random"`) with an `if/elif` chain into a version that accepts a `key`/strategy function instead.
   *Hint: Python's first-class functions often make a full Strategy class hierarchy unnecessary.*

3. **Kill the Singleton** — Given a `ConfigSingleton` class used globally throughout a codebase, rewrite the dependent functions to accept a `config` parameter instead (dependency injection), and explain what testing benefit this gives you.
   *Focus: identify why the Singleton made unit testing awkward (shared global state between tests).*

4. **Adapter for a third-party API** — Design (interfaces + a couple of methods, no need for a full implementation) an `EmailSender` protocol and two adapters wrapping two different fictional email providers with different method signatures.
   *Hint: your calling code should be able to swap providers by swapping the adapter, with zero changes elsewhere.*

5. **Decorator pattern vs syntax** — Implement a `LoggingRepository` class that wraps any object implementing a `Repository` protocol and logs every call before delegating to the wrapped object, without modifying the wrapped class.
   *Focus: distinguish this "wrap and delegate" object composition from the `@decorator` function syntax from Chapter 1.*

6. **Entities vs value objects** — Given a `Money` concept (amount + currency) and an `Order` concept (has an ID, a status, a list of line items), decide which should be a value object and which an entity, and justify each with an equality argument.
   *Hint: ask "if I create two instances with identical field values, should they be treated as the same thing?"*

7. **Extract an interface (DIP)** — Given a `NotificationService` class that directly imports and calls a specific SMS provider's SDK inline in its methods, refactor it to depend on an abstract `SmsGateway` Protocol instead, injected via the constructor.
   *Focus: after refactoring, no import of the concrete SDK should appear inside `NotificationService` itself.*

8. **Repository pattern from scratch** — Design a `UserRepository` Protocol (`add`, `get_by_id`, `get_by_email`) and implement two versions: an in-memory dict-backed one and a stub that would wrap a real DB (signatures only, `...` bodies are fine for the DB one).
   *Hint: write a small test suite against the Protocol that both implementations pass — that's the point of the pattern.*

9. **Refactor a god object** — Given a `UserManager` class that handles registration, password hashing, sending welcome emails, and generating usage reports, split it into at least three classes with single, clear responsibilities, wired together via composition.
   *Focus: name each new class after its one responsibility; if you struggle to name it precisely, it's probably still doing too much.*

10. **Replace conditional with polymorphism** — Given a function that computes a shipping cost via a long `if shipping_type == "standard": ... elif shipping_type == "express": ...` chain, refactor it into a small set of classes (or strategy functions) implementing a common `calculate(order) -> float` interface.
    *Hint: adding a new shipping type afterward should require adding new code, not editing the existing chain (Open/Closed Principle).*

11. **Hexagonal slice** — Take a small function that both queries a SQLite database directly (via raw `sqlite3`) and applies a business rule (e.g., "flag overdue invoices"), and split it into a repository (DB access only), a pure domain function (the business rule, no DB import), and a thin orchestration function that wires them together.
    *Focus: the domain function should be testable by passing in plain Python objects, with zero DB or mocking involved.*

12. **Swap the adapter under test** — For the hexagonal slice from problem 11, write two versions of the same test (one using a real SQLite-backed repository, one using an in-memory fake repository) asserting identical behavior, proving the domain logic doesn't care which one is used.
    *Hint: this is the exact proof-of-decoupling exercise TaskFlow's own Chapter 8 project asks for — do it here first on a small scale.*
