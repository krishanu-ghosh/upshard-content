---
title: "Singleton and Factory Method"
description: One guaranteed instance versus one hidden decision about which concrete class to build — and why Singleton's convenience comes with a real cost.
roadmap: software-design-architecture
node: singleton-and-factory-method
group: creational-patterns
video:
  provider: youtube
  id: "p_UQ7tzUFLo"
  start: 0
  end: null
creator:
  name: ArjanCodes
  channel_url: https://www.youtube.com/@ArjanCodes
duration_seconds: 930
level: intermediate
prerequisites: [test-driven-development]
takeaways:
  - "Singleton guarantees that a class has exactly one instance and gives the whole program one shared point to reach it from, usually through a static `getInstance()` method and a private constructor that blocks anyone else from calling `new` directly."
  - "The real cost of Singleton is global mutable state: any code anywhere can read or change the shared instance, so a bug caused by one call site mutating it can surface in a completely unrelated part of the program."
  - "Singleton also hides a dependency — a class that calls `ConfigLoader.getInstance()` inside a method never declares that dependency in its constructor, so a test can't swap in a fake config without reaching into global state."
  - "Most modern codebases get the same 'one shared instance' benefit from dependency injection instead: construct the instance once, then pass it explicitly to whatever needs it, which keeps the dependency visible and swappable in tests."
  - "Factory Method solves a different problem: which concrete class gets built, not how many instances exist. A base class defines a method like `createDocument()` that it calls internally but never implements, leaving each subclass to override it and return its own concrete type."
  - "Because the decision of which class to instantiate lives in an overridable method, adding a new product type means writing a new subclass, not editing a growing `if/else` chain inside code that already works."
check_questions:
  - q: Why does a Singleton accessed via a static method make unit testing harder?
    a: The dependency is invisible from the outside — a class that calls `getInstance()` inside its own methods never lists that dependency in its constructor — so a test can't hand it a fake or mock version without monkey-patching the global instance itself, and any two tests that run in the same process share the exact same live object.
  - q: How does Factory Method let a codebase support a new product type without editing existing code?
    a: The base class only ever calls an overridable creation method like `createDocument()` and never implements it; adding support for a new type means writing a new subclass that overrides that method and returns the new concrete class, leaving the base class and every other subclass untouched.
  - q: What's the actual difference between what Singleton and Factory Method each control?
    a: Singleton controls quantity — it guarantees exactly one instance of a specific class exists and is reachable globally. Factory Method controls identity — it hides which concrete class gets built behind a method call, independent of how many instances of anything get created.
deeper:
  - label: "The Factory Pattern in Python // Separate Creation From Use"
    url: "https://www.youtube.com/watch?v=s_4ZrtQs8Do"
related_formulas: []
---

## Two different questions about object creation

Most of the time, calling a constructor directly is fine — the caller knows exactly which class it wants, and there's no reason to need more than one. Two situations break that assumption. Sometimes a type genuinely needs to exist exactly once, shared everywhere it's used — a single connection pool, a single application-wide logger. Other times, the code that needs an object doesn't actually know, or shouldn't have to know, which concrete class it's getting — only the specific context a subclass has access to can decide that. Singleton answers the first question, "how many." Factory Method answers the second, "which one." They get taught together because they're both about controlling instantiation, but conflating them is the usual mistake — a Singleton isn't a Factory Method just because both involve a special creation method.

## Singleton: one instance, reachable from anywhere

The textbook shape is a class with a private constructor — so nothing outside the class can call `new` — and a static method, usually `getInstance()`, that creates the instance the first time it's called and returns that same object every time after. Before this pattern, a naive `ConfigLoader` gets instantiated independently in a dozen different files, each one re-parsing the same config file from disk, with no guarantee that every copy agrees. After, every call site asks `ConfigLoader.getInstance()` and gets the one object that was ever built.

Here's the part worth being honest about: Singleton is one of the most criticized patterns in the original catalog, and a lot of experienced engineers treat it as an anti-pattern by default. The problem isn't the "one instance" guarantee — it's the global, static access point that comes with it. Any class, anywhere, can reach into `ConfigLoader.getInstance()` without that dependency ever showing up in its constructor signature. That makes the dependency invisible to anyone reading the class's public interface, and it makes testing painful — a test can't swap in a fake config without reaching into shared global state, and tests that run in the same process can quietly affect each other through it. A static global is also just a global, no matter how disciplined the access method looks.

The usual modern fix is dependency injection: build the single instance once, near the program's entry point, and hand it explicitly to whatever constructor needs it. The "one shared instance" property survives; the hidden, static, globally-reachable part doesn't. Singleton still earns its place for things that are genuinely process-wide and stateless by nature — a thread pool, a hardware resource handle — but reaching for it as a default way to share an object is usually solving the wrong problem.

## Factory Method: deferring the "which class" decision

Picture an `Application` class that needs to produce documents, and a method on it, `newDocument()`, that currently reads `if (docType == "resume") return new Resume(); else if (docType == "report") return new Report(); ...`. Every time the business adds a new document type, this method gets another branch, and every existing branch has to be re-read to make sure the new one doesn't collide with it.

Factory Method restructures this by making the creation step itself overridable. `Application` declares `newDocument()` as a method whose body calls `createDocument()` — an abstract method it never implements. A `ResumeApplication` subclass overrides `createDocument()` to return `new Resume()`; a `ReportApplication` subclass overrides it to return `new Report()`. The code in `newDocument()` that does something useful with the result — opening it, adding it to a recent-files list — never needs to know or check which concrete type it got back. Adding a new document type means writing a new subclass that overrides one method; nothing that already works gets touched or re-tested.

## Where each one stops paying for itself

Factory Method costs a class per variant — if there are only ever going to be two document types and that's not changing, the `if/else` it replaces was never the real problem. And Singleton's convenience is exactly what makes it easy to reach for by habit; the moment a "shared instance" needs to be different in a test, or swapped for a second configuration in the same process, the global access point that looked convenient becomes the thing blocking the fix. Both patterns are about taking a creation decision out of ad-hoc code and putting it somewhere disciplined — but it's worth checking, each time, whether that decision actually needs to move at all.
