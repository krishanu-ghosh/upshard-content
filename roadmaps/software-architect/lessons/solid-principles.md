---
title: "SOLID Principles"
description: The five object-oriented design principles — SRP, OCP, LSP, ISP, DIP — with a concrete before/after code shape for each one.
roadmap: software-architect
node: solid-principles
group: principles-and-patterns
video:
  provider: youtube
  id: "E4UQW0pn4nE"
  start: 0
  end: null
creator:
  name: Design Patterns Lectures
  channel_url: https://www.youtube.com/@DesignPatternsLectures
duration_seconds: 478
level: intermediate
prerequisites: []
takeaways:
  - "SOLID is five heuristics for keeping a change in one part of a codebase from forcing changes everywhere else — not a checklist to satisfy for its own sake."
  - "Single Responsibility means a class has one reason to change; a class that validates input, sends email, and saves to a database has three, and a change to any one of them risks breaking the others."
  - "Open/Closed means new behavior gets added by writing a new class that implements an existing interface, not by editing a method's if/else chain every time a new case shows up."
  - "Liskov Substitution means any code written against a base type must keep working, unmodified, if a subtype is swapped in — a Square that extends Rectangle but overrides both setWidth and setHeight to stay square breaks every test written against Rectangle's contract."
  - "Interface Segregation means a class should never be forced to implement a method it has no use for just because it shares an interface with classes that do."
  - "Dependency Inversion means both a high-level policy and a low-level detail depend on an interface that sits between them, so the policy never has to change when the detail does."
check_questions:
  - q: Why does a class with three unrelated responsibilities violate SRP even if it works correctly today?
    a: Each responsibility is a separate reason the class might need to change in the future — a new validation rule, a new email provider, a new database — and any one of those changes risks breaking the other two responsibilities that happen to live in the same class.
  - q: What's the standard fix for an if/else chain over a type field that grows every time a new case is added?
    a: Replace the if/else with a shared interface and one class per case — the Strategy pattern — so adding a new case means adding a new class that implements the interface, never editing the code that already works.
  - q: How does the classic Square-extends-Rectangle example violate Liskov Substitution?
    a: Rectangle's contract lets width and height vary independently, but Square has to keep them equal, so overriding setWidth to also change height breaks any code written against Rectangle that expects setting width alone to leave height untouched — the subtype can't be substituted for the base type without changing behavior.
deeper:
  - label: "I Tried SOLID Principles in Python… Here's What Happened"
    url: "https://www.youtube.com/watch?v=uxwjXLjJOoM"
related_formulas: []
---

## Five heuristics, not five laws

SOLID is an acronym for five object-oriented design principles that Robert C. Martin collected and named in the early 2000s: Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, and Dependency Inversion. None of them is a rule a compiler enforces — they're heuristics aimed at one practical problem: keeping a change to one part of a system from rippling, unpredictably, into parts that had nothing to do with it. Applied with judgment, they make a codebase easier to extend and test. Applied dogmatically — wrapping every function in an interface "just in case," splitting a ten-line class into four one-line classes — they add indirection that costs more than the flexibility is worth. The point of learning them is knowing which trade-off you're making, not satisfying a checklist.

## Single Responsibility: one reason to change

A class has a single responsibility when there's exactly one reason it would ever need to be edited. A `UserService` that validates email format, sends a welcome email, and writes a row to the database has three reasons to change: a new validation rule, a new email provider, a new database schema. Worse, those three concerns are now coupled — a test for validation has to construct a database connection it doesn't need, and a bug in the email-sending code can't be fixed without redeploying the part that also validates input. Splitting it into `UserValidator`, `WelcomeEmailSender`, and `UserRepository` doesn't reduce the total amount of code; it makes each piece independently testable, independently reusable, and independently safe to change.

## Open/Closed: add behavior without editing what works

A module is open for extension but closed for modification when new behavior is added by writing new code, not by editing code that already works and already has tests passing against it. The usual violation is an `if/else` or `switch` over a type field — `if (customerType == "gold") applyGoldDiscount(); else if (customerType == "silver") ...` — that grows a new branch every time the business adds a tier. The fix is the Strategy pattern: define a `DiscountStrategy` interface with one `apply(order)` method, write one class per tier, and let whatever constructs the order pick the right strategy. Adding a platinum tier now means adding a `PlatinumDiscountStrategy` class; the code that applies a discount never changes again, so it can't regress.

## Liskov Substitution: a subtype has to keep every promise the base type made

Named for Barbara Liskov, this principle says that anywhere a function accepts a base type, substituting any subtype must not break that function's behavior. The textbook violation is `Square extends Rectangle`: Rectangle's contract is that `setWidth` and `setHeight` are independent, but a Square has to keep both sides equal, so its `setWidth` override silently changes the height too. Any test written against Rectangle — "set width to 5, height stays whatever it was" — now fails the moment a Square is substituted in, even though Square is a geometrically valid subtype of Rectangle. The lesson isn't about geometry; it's that inheritance should model "is substitutable for," not "is conceptually a kind of."

## Interface Segregation: don't force a class to implement what it can't do

A fat interface — `interface Worker { work(); eat(); sleep(); }` — forces every implementer to provide all three methods, even one that logically can't do one of them. A `RobotWorker` has to implement `eat()` with an empty body or a thrown exception, either of which is a lie about what the class actually does and a trap for any caller that assumes `eat()` does something. Splitting the fat interface into `Workable`, `Feedable`, and `Restable` lets `RobotWorker` implement only `Workable`, so its public contract is honest about what it supports.

## Dependency Inversion: both sides depend on an interface, not on each other

High-level policy (an `OrderProcessor`) shouldn't depend directly on low-level detail (a `PostgresOrderRepository`), and the detail shouldn't be what the policy is written against — both should depend on an interface, `OrderRepository`, that sits between them. This is the principle; dependency injection, covered in this roadmap's `architectural-patterns-and-dependency-injection`, is the mechanical technique that satisfies it in code, by having something outside the class decide which concrete implementation to hand over. Confusing the two is common: inversion is "depend on abstractions," injection is one way of making that happen at runtime.

## Where SOLID earns its keep, and where it doesn't

These five principles pull in the same direction — toward smaller, more substitutable, more independently-testable units — but every one of them has a cost in extra types and extra indirection. A ten-line script that runs once doesn't need a `Strategy` interface; a class with one genuine, stable responsibility doesn't need to be split further just to look SOLID on a diagram. The skill this lesson is actually teaching is recognizing *when* a piece of code is about to change in ways that will hurt, and reaching for the relevant principle only then — not applying all five uniformly to every class in a codebase regardless of whether it's likely to change at all.
