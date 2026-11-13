---
title: "Architectural Patterns & Dependency Injection"
description: Layered architecture, MVC/MVP/MVVM, and hexagonal architecture, plus dependency injection as the technique that makes them work.
roadmap: software-architect
node: architectural-patterns-and-dependency-injection
group: principles-and-patterns
video:
  provider: youtube
  id: "eHhXoCNCI1c"
  start: 0
  end: null
creator:
  name: ArjanCodes
  channel_url: https://www.youtube.com/@ArjanCodes
duration_seconds: 1466
level: intermediate
prerequisites:
  - solid-principles
takeaways:
  - "An architectural pattern operates at the scale of the whole application — where UI code lives, where business logic lives, where data access lives — rather than at the scale of one class relationship the way a Gang of Four pattern does."
  - "MVC splits an app into a Model (data and business rules), a View (what the user sees), and a Controller (translates user input into changes to the model), with MVP and MVVM each moving the line for how much the View is allowed to know on its own."
  - "Hexagonal (also called ports-and-adapters or clean) architecture's one real idea is that dependencies point inward: the business logic sits in the center knowing nothing about databases or web frameworks, and adapters at the edges translate between the outside world and the core."
  - "The specific labels — hexagonal, onion, clean — disagree on how many rings to draw, but they all enforce the same rule: outer, infrastructure-heavy layers may depend on inner, business-logic layers, never the other way around."
  - "Dependency injection is how \"depend on an interface, not a concrete class\" gets implemented in practice — a class declares what it needs in its constructor, and something else (a caller, or a DI framework) decides what concrete object to hand it."
check_questions:
  - q: What's the core difference between an architectural pattern and a design pattern like Strategy or Decorator?
    a: A design pattern shapes the relationship between a handful of objects or classes; an architectural pattern shapes how the entire codebase is divided into layers or components, deciding where categories of code — UI, business logic, data access — are allowed to live.
  - q: In hexagonal architecture, why does the business logic at the center never import a database driver directly?
    a: Hexagonal architecture requires dependencies to point inward — the core defines an interface (a "port") describing what it needs, like `UserRepository.save(user)`, and an outer "adapter" class implements that interface using a real database; the core only ever depends on its own interface, never on the concrete database library.
  - q: How does dependency injection relate to "program to an interface, not an implementation"?
    a: Dependency injection is the mechanism that makes that advice practical — a class's constructor asks for an interface type, and whoever constructs the object (by hand, or via a DI framework reading configuration) decides which concrete implementation to pass in, so the class itself never hard-codes a specific dependency.
deeper:
  - label: "Hexagonal vs Clean vs Onion Architecture... It Doesn't Matter"
    url: "https://www.youtube.com/watch?v=Thzz5bvUI40"
  - label: "Dependency INVERSION vs Dependency INJECTION in Python"
    url: "https://www.youtube.com/watch?v=2ejbLVkCndI"
related_formulas: []
---


## From object relationships to application shape

The Gang of Four's object-oriented patterns — Factory, Decorator, Strategy, and the rest (see `computer-science`'s `gof-design-patterns`) — each describe a relationship between a small number of classes. An architectural pattern operates one level up: it decides how an entire codebase gets divided into layers or components, and which layers are allowed to know about which others. Get this wrong and a codebase ends up with business rules scattered through database queries and UI event handlers, so that changing how discounts are calculated means hunting through a dozen files that each embed a slightly different copy of the logic. Architectural patterns exist to keep that one calculation in one place.

## Layered architecture and the MV* family

The oldest and simplest answer is a **layered architecture**: a presentation layer, a business-logic layer, and a data-access layer, stacked so each layer only talks to the one directly below it. The **MV\*** family of patterns is a layered architecture specialized for applications with a user interface. **MVC** (Model-View-Controller) splits the world into a Model holding data and business rules, a View that renders it, and a Controller that takes user input and decides how the Model should change. The three variants in this lesson's video — MVC, **MVP** (Model-View-Presenter), and **MVVM** (Model-View-ViewModel) — all keep the same Model and View, and differ mainly in how much the View is trusted to do on its own. In classic MVC the View can read directly from the Model; MVP inserts a Presenter that the View talks to exclusively, so the View never touches the Model directly and becomes easier to test by substituting a fake Presenter; MVVM goes further by having the View **bind** to a ViewModel's properties, so changes to the ViewModel's state propagate to the View automatically through the UI framework's data-binding system, rather than the View polling or being explicitly told to re-render.

## Hexagonal architecture: one idea wearing several names

**Hexagonal architecture** (also called **ports-and-adapters**, and closely related to **clean architecture** and **onion architecture**) starts from a different question: what if the database and the web framework are the *least* important parts of the system, and the business rules are the most important? Hexagonal puts the business logic — the "domain" — at the dead center, with no dependency on anything outside it. The domain defines **ports**: interfaces describing what it needs, like `OrderRepository.save(order)` or `PaymentGateway.charge(amount)`. Outside that center sit **adapters** — concrete classes that implement those ports using a real Postgres database, a real Stripe API, a real HTTP framework. The domain never imports the database driver or the web framework; it only knows about its own interfaces, and adapters are plugged in at the edges to satisfy them. Swap Postgres for MongoDB, or REST for GraphQL, and in principle only the adapter layer changes — the domain logic doesn't notice.

The names hexagonal, onion, and clean architecture disagree on how many concentric rings to draw and what to call them, but they share the one rule that actually matters: **dependencies point inward**. An outer, infrastructure-heavy layer is allowed to depend on an inner, business-logic layer; the reverse is never allowed. Arguing about whether a given diagram has three rings or four, or whether something is called a "use case" or a "service," misses that the dependency direction is the entire point — which is exactly the critique this lesson's deeper video on hexagonal vs. clean vs. onion makes.

## Dependency injection: how the inward-pointing rule is enforced in code

All of this inward-pointing-dependency talk is abstract until a class actually needs a database to save an order. If the `Order` class's code says `new PostgresOrderRepository()` directly, it has just created a hard dependency on Postgres, breaking the rule. **Dependency injection** is the fix: instead of constructing its own dependencies, a class declares what it needs through its constructor — `class OrderService { constructor(repository: OrderRepository) { ... } }` — using the interface type, not a concrete class. Something *else* decides which concrete implementation to hand over: that might be a few lines of manual wiring in a small app (`new OrderService(new PostgresOrderRepository())`), or a dependency-injection framework in a larger one, reading configuration to decide. Either way, `OrderService` itself never imports Postgres — it only knows the shape of `OrderRepository`, which is precisely what lets hexagonal architecture's domain layer stay ignorant of what's plugged into its ports. This is also why dependency injection and dependency *inversion* get confused: inversion is the principle (depend on abstractions, not concretions), and injection is one common technique for satisfying it, which is the exact distinction the deeper video on DI vs. inversion walks through.
