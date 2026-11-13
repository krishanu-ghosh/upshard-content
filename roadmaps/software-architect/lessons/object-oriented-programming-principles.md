---
title: "The Four Pillars of Object-Oriented Programming"
description: Encapsulation, abstraction, inheritance, and polymorphism — what each one actually buys you, not just its definition.
roadmap: software-architect
node: object-oriented-programming-principles
group: paradigms
video:
  provider: youtube
  id: "pTB0EiLXUC8"
  start: 0
  end: null
creator:
  name: Programming with Mosh
  channel_url: https://www.youtube.com/@programmingwithmosh
duration_seconds: 454
level: beginner
prerequisites: []
takeaways:
  - "Encapsulation means bundling data with the methods that operate on it and hiding the data behind a controlled interface, so a class's internal representation can change without breaking every caller that uses it."
  - "Abstraction means exposing only what a consumer needs to know to use something — a method name and its contract — while hiding how it's implemented, which is encapsulation applied at the level of an entire module or interface."
  - "Inheritance lets a subclass reuse a superclass's fields and methods and override the ones that should behave differently, but it creates a tight coupling between the two classes that composition usually avoids."
  - "Polymorphism lets code call the same method name on objects of different types and get type-appropriate behavior back, which is what lets a function written against a base type work unmodified against any subtype."
  - "Favoring composition (an object holding a reference to another) over inheritance (an object being a specialized version of another) is a decades-old piece of advice precisely because inheritance hierarchies get brittle fast once they're more than two levels deep."
check_questions:
  - q: Why does encapsulation matter even in a single-developer codebase with no "attackers" trying to misuse the class?
    a: It isn't about attackers — it's about change. If a class's fields are public, every caller can come to depend on their exact current representation, and changing that representation later breaks all of them; a controlled interface lets the internals change freely as long as the interface's contract stays the same.
  - q: What's the practical difference between abstraction and encapsulation?
    a: Encapsulation is the mechanism — bundling data with its methods and restricting direct access to the data; abstraction is the result — a consumer sees a simple interface and doesn't need to know or care how it's implemented underneath.
  - q: Why can inheritance become a liability as a codebase grows?
    a: A deep inheritance chain means a change to a base class can silently change behavior several levels down in ways that are hard to trace, and a subclass is forced to inherit everything from its parent even the parts it doesn't want, which composition avoids by only pulling in exactly the behavior a class actually needs.
related_formulas: []
---

## Why these four ideas get bundled together

Object-oriented programming is usually introduced as "encapsulation, abstraction, inheritance, polymorphism" as if the four were a package deal, but they answer four different questions, and a working architect needs to be able to pull them apart. Encapsulation and abstraction are about hiding: what does a piece of code let the outside world see, and what does it keep to itself? Inheritance and polymorphism are about reuse and substitution: how does one piece of code build on another, and how does calling code stay indifferent to exactly which concrete type it's handed? Keeping these two pairs distinct in your head is what lets you reason about a design instead of just reciting its vocabulary.

## Encapsulation: data and behavior, bundled and guarded

A class bundles state (fields) with the operations that are allowed to touch that state (methods), and it controls access to that state from the outside — typically by making fields private and exposing only the methods that mutate them in valid ways. A `BankAccount` class doesn't expose its `balance` field directly for anyone to set to a negative number; it exposes `withdraw(amount)`, which can refuse the operation if the balance would go negative. The payoff isn't security against malicious code in most codebases — it's change management. As long as `withdraw`'s contract stays the same ("never leaves the balance negative"), the class is free to change how it stores or computes that balance internally — switching from a simple integer to a ledger of transactions, say — without touching a single caller.

## Abstraction: the interface is the product

Abstraction is encapsulation's effect scaled up to an entire module, class, or interface: a consumer only ever needs to know a method's name, its inputs, and what it promises to return — never how it gets there. A `PaymentProcessor.charge(amount)` interface might be backed by Stripe today and a different provider tomorrow; code that calls `charge()` never needs to change, because the abstraction — "something that can charge an amount and tell you if it succeeded" — is the actual contract being programmed against, not any one implementation of it. This is the same idea `architectural-patterns-and-dependency-injection`'s hexagonal architecture applies at the scale of a whole system: the core depends on an abstraction (a port), and a concrete adapter satisfies it.

## Inheritance: reuse through "is-a"

Inheritance lets a class declare that it's a specialized version of another: a `SavingsAccount` *is a* `BankAccount` with an added interest rate, so it inherits `withdraw()` and adds or overrides whatever makes it different. This is a legitimate and useful tool when the "is-a" relationship is genuinely true and stable — a `Circle` really is a `Shape` in every sense the program cares about. The trouble starts when inheritance gets reached for as a reuse mechanism rather than a genuine type relationship: a `ReportGenerator` that inherits from `DatabaseConnection` just to reuse its connection-handling code isn't modeling an "is-a" relationship at all, and now every change to `DatabaseConnection` is a change the report generator has to absorb whether it wants to or not.

## Polymorphism: one call, many behaviors

Polymorphism is what makes inheritance (or an implemented interface) actually pay off at the call site: code that has a reference to a `Shape` can call `shape.area()` without knowing or caring whether the concrete object is a `Circle`, a `Square`, or a `Triangle` — each overrides `area()` with its own formula, and the right one runs. This is what lets you write a function like `totalArea(shapes: Shape[])` once and have it keep working correctly forever as new shapes get added, as long as each new shape correctly implements `area()`. Without polymorphism, that function would need an `if/else` chain checking the concrete type of every shape — exactly the kind of code that has to be edited every time a new type is added, which is the thing object-oriented design exists to avoid.

## Composition over inheritance, as a default

The video's running theme — and the field's hard-won consensus since the 1990s "Gang of Four" book formalized it as a principle — is that composition should be your default and inheritance the exception you reach for only when the "is-a" relationship is real and stable. A `Car` that *has an* `Engine` (composition: the `Car` holds a reference to an `Engine` object and delegates to it) is far more flexible than a `Car` that somehow *is an* `Engine`: you can swap in a different engine implementation at runtime, test the car with a fake engine, and avoid the car inheriting any accidental behavior the engine class happens to expose. Keep this ordering in mind heading into `solid-principles` next: several of those five principles — especially the Liskov Substitution Principle and the Dependency Inversion Principle — are really just sharper, more enforceable versions of the inheritance and polymorphism caveats raised here.
