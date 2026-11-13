---
title: "GoF Design Patterns"
description: The Gang of Four's creational, structural, and behavioral patterns, with a deep look at Strategy plus the Null Object and Type Object patterns.
roadmap: computer-science
node: gof-design-patterns
group: design-patterns-and-architecture
video:
  provider: youtube
  id: "v9ejT8FO-7I"
  start: 0
  end: null
creator:
  name: Christopher Okhravi
  channel_url: https://www.youtube.com/@ChristopherOkhravi
duration_seconds: 2111
level: intermediate
prerequisites: []
takeaways:
  - "The 1994 \"Gang of Four\" book grouped reusable object-oriented solutions into three families by what problem they solve: creational (how objects get made), structural (how objects are composed into larger pieces), and behavioral (how objects communicate and share responsibility)."
  - "A creational pattern like Factory Method hides which concrete class gets instantiated behind a method call, so callers depend on an interface instead of a constructor."
  - "A structural pattern like Decorator wraps an object in another object implementing the same interface, adding behavior without touching the original class or spawning a combinatorial subclass tree."
  - "The Strategy pattern swaps the Gang of Four's classic alternative to a long if/else chain — one algorithm family, each variant behind the same interface, chosen and swapped in at runtime."
  - "Null Object and Type Object aren't in the original 1994 catalog, but they solve the same kind of recurring problem: Null Object replaces defensive null-checks with a do-nothing object of the right type, and Type Object turns \"add a new type\" into \"add a new data row\" instead of a new class."
check_questions:
  - q: What distinguishes a creational pattern from a structural pattern?
    a: A creational pattern controls how an object gets created (which concrete class, when, and by whom), while a structural pattern controls how already-created objects are composed into larger structures without changing their own code.
  - q: Why does the Strategy pattern beat a long if/else or switch statement for choosing between algorithms?
    a: Each branch of an if/else has to be edited in place every time a new case is added, and the whole block lives in one class; Strategy puts each algorithm behind the same interface in its own class, so adding a new one means adding a new class rather than editing existing code.
  - q: How does the Null Object pattern remove the need for `if (thing != null)` checks scattered through a codebase?
    a: Instead of a reference being null when there's "nothing there," it points at a real object of the same type whose methods are safe no-ops (return zero, do nothing, log nothing), so callers can invoke methods on it exactly like a normal instance without special-casing the absence first.
  - q: When does the Type Object pattern pay off over just adding new subclasses?
    a: When a game or app keeps adding near-identical "kinds of thing" — monster types, item types, enemy tiers — each needing only a few different field values rather than different behavior; Type Object holds that variation as data in instances of one class (and its companion "type" class) instead of multiplying subclasses for every new kind.
deeper:
  - label: "Null Object Pattern – Design Patterns (ep 18)"
    url: "https://www.youtube.com/watch?v=rQ7BzfRz7OY"
  - label: "Prototype & Type Object Pattern - Game Programming Patterns in Unity & C#"
    url: "https://www.youtube.com/watch?v=x00IewDdrUA"
related_formulas: []
---


## Why a catalog of patterns exists at all

In 1994, four authors — Gamma, Helm, Johnson, and Vlissides, forever shortened to "the Gang of Four" — published a catalog of 23 solutions to problems that kept showing up in object-oriented code, regardless of the domain. None of them invented these solutions from scratch; the book's contribution was naming them, so that "use the thing where you wrap an object to add behavior without subclassing it" became "use Decorator," and two engineers who'd never worked together could agree on a design in one sentence. The patterns split into three families by what they're solving. **Creational** patterns control how objects come into existence, **structural** patterns control how objects are wired together into bigger pieces, and **behavioral** patterns control how objects talk to each other and share work.

## Creational: controlling how things get built

The simplest creational problem is "I don't want the caller to know which concrete class it's getting." A **Factory Method** solves this by putting object construction behind a method — `createShape()` returns something that implements the `Shape` interface, and the caller never writes `new Circle()` or `new Square()` directly. This matters once a codebase has many call sites that create shapes: changing which subclass gets built, or adding validation before construction, happens in one place instead of everywhere `new` was typed. **Singleton** — ensuring a class has exactly one instance, reachable from anywhere — is the catalog's most famous creational pattern and also its most argued-over, since a global mutable instance makes testing harder. Most modern code reaches for dependency injection (the next lesson) to get the same "one shared instance" property without a true global.

## Structural: composing without subclassing

Structural patterns answer "how do I combine objects into something bigger without making an unmanageable inheritance tree?" **Adapter** wraps an object with an incompatible interface in a new class that translates calls into something the rest of the system understands. The classic example is wrapping a third-party payment library's `charge(amount_cents)` method behind your own `PaymentProcessor.pay(amount)` interface, so the rest of the app never touches the vendor's shape directly. **Decorator** solves a different problem: adding behavior to an object at runtime without touching its class. A `Coffee` object can be wrapped in a `MilkDecorator`, then a `SyrupDecorator`, each implementing the same `Beverage` interface and delegating to the one it wraps before adding its own cost and description. The alternative — a subclass for every combination of milk and syrup — grows combinatorially.

## Behavioral: Strategy as the running example

Behavioral patterns govern how responsibility is shared and how objects communicate. The one this lesson's video goes deep on is **Strategy**: you have a family of interchangeable algorithms — say, three ways to sort a list, or three ways to validate a password — and instead of a method with a long `if`/`else if`/`else` picking between them, each algorithm gets its own class implementing a shared `Strategy` interface with one method, like `execute()`. The object that needs the behavior (the "context") holds a reference to whichever strategy it was configured with and calls `strategy.execute()` without knowing or caring which concrete implementation that is. Swapping behavior at runtime becomes "assign a different strategy object" instead of "edit the branch logic," and adding a new variant becomes "write a new class" instead of "add another `else if` to an already-long method that every variant shares." Other behavioral patterns follow the same spirit: Observer lets objects subscribe to be notified of another object's changes, and Command wraps a request as an object so it can be queued, logged, or undone. Both pull a piece of varying behavior out into its own object so the rest of the system can stay stable while that piece changes.

## Two patterns outside the original 23

**Null Object** isn't in the 1994 book, but it's one of the most practically useful patterns to know. The problem: a method returns either a real object or `null` to mean "nothing here," and every caller has to remember to check for `null` before calling a method on the result — forget one check, and the program crashes. The fix is to return a real object of the same type instead, one whose methods are safe no-ops: a `NullLogger.log(message)` that does nothing, a `NullCustomer.getDiscount()` that returns zero. Callers stop needing the `if (x != null)` guard entirely, because there's always a valid object to call methods on — it's just one that does nothing.

**Type Object** solves a problem that shows up constantly in games and content-heavy apps: you keep adding new "kinds" of something — new monster types, new item types — and each kind differs only in a handful of field values (health, damage, icon), not in actual behavior. Subclassing for every kind means a new class file for every new monster, most of which is copy-pasted boilerplate. Type Object instead has one `Monster` class holding a reference to a `MonsterType` object, and `MonsterType` holds the data that varies (name, base health, base damage). Adding a new monster kind becomes "create a new `MonsterType` instance with different field values" — often loadable straight from a data file — rather than "write and compile a new class." The trade is that behavior differences between kinds, not just data, are harder to express this way. That's exactly when a Gang of Four pattern like Strategy, or genuine subclassing, becomes the better tool again.
