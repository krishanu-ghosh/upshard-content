---
title: "Observer and Strategy"
description: Observer lets objects subscribe to state changes without knowing each other's types; Strategy swaps an algorithm's implementation behind one shared interface.
roadmap: software-design-architecture
node: observer-and-strategy
group: behavioral-patterns
video:
  provider: youtube
  id: "-oLDJ2dbadA"
  start: 0
  end: null
creator:
  name: Geekific
  channel_url: https://www.youtube.com/@geekific
duration_seconds: 408
level: intermediate
prerequisites: [composite-bridge-and-flyweight]
takeaways:
  - "Observer and Strategy both exist to pull behavior that varies out of an object that would otherwise have to know about every version of it — Observer for who gets told when a change happens, Strategy for which algorithm runs right now."
  - "In Observer, a subject keeps a list of objects implementing a shared `Observer` interface and calls `update()` on each one when its own state changes, instead of hardcoding calls to specific dependent objects by name."
  - "A spreadsheet is the clearest Observer example: a cell holding a formula subscribes to every cell it reads from, so when one of those cells changes, it gets notified and recalculates — the changed cell never needs to know which formulas depend on it."
  - "Strategy replaces a growing `if/else` or `switch` over algorithm choice with one interface — say `DiscountStrategy` — and one class per variant, so the object using the algorithm (the \"context\") just calls `strategy.apply()` without caring which concrete class it's holding."
  - "Both patterns can misfire: Observer notifications can cascade into update storms or silently leak memory when a subscriber forgets to unsubscribe, and Strategy adds a class and an interface for behavior that might never actually need a second variant."
  - "The two patterns compose naturally — an Observer's `update()` method commonly delegates to a Strategy object to decide *how* to react, keeping \"who gets notified\" and \"what happens next\" as two separate, independently swappable decisions."
check_questions:
  - q: Why doesn't a spreadsheet cell need to know which formulas depend on it when Observer is used correctly?
    a: The cell only maintains a list of subscribed observers that implement a shared interface and calls a generic `update()` method on each one — it never references a specific formula's class or variable name, so new formulas can subscribe or unsubscribe without the cell's code changing at all.
  - q: What's the standard fix for an `if/else` chain over algorithm choice, and what does Strategy cost you in return?
    a: Strategy replaces the chain with one interface and one class per algorithm variant, so adding a new variant means adding a new class instead of editing a method everyone depends on — the cost is an extra interface and an extra class per variant, which is wasted ceremony if the algorithm is never actually going to vary.
  - q: What failure mode is unique to Observer that Strategy doesn't share?
    a: Observer introduces a notification chain — if an observer's `update()` method itself changes state on something else being observed, you get a cascade that's hard to trace, and if a subscriber is destroyed without calling `unsubscribe()`, the subject keeps a dangling reference that leaks memory; Strategy has neither problem, since a strategy object doesn't react to anything, it's just called directly.
deeper:
  - label: "The Strategy Pattern Explained and Implemented in Java | Geekific"
    url: "https://www.youtube.com/watch?v=Nrwj3gZiuJU"
related_formulas: []
---

## Two ways to keep behavior from hardening into one object

Both of these patterns solve the same underlying problem from different angles: something in your code needs to vary — who reacts to a change, or how a calculation gets done — and you don't want that variation baked into one method that grows a new branch, a new field, or a new hardcoded call every time a new case shows up. Observer decouples a subject from the list of things that react to it. Strategy decouples an algorithm's caller from which specific implementation it's running. Neither is exotic once you see the shape: an interface sits between two things that would otherwise know too much about each other.

## Observer: a subject that doesn't know who's listening

Picture a spreadsheet. Cell `C1` holds `=A1+B1`. The naive way to make this work is for `A1` and `B1` to each keep a hardcoded reference to `C1` and call `C1.recalculate()` directly whenever they change. That works until a second formula, `D1`, also reads from `A1` — now `A1`'s change-handling code has to know about `C1` *and* `D1` by name, and every new formula means editing `A1`'s code again.

Observer flips this. `A1` becomes a **subject**: it holds a list of objects implementing an `Observer` interface with one method, `update()`. Any cell that depends on `A1` calls `A1.subscribe(this)` once, and `A1` never again needs to know what kind of thing it's notifying — it just loops over its subscriber list and calls `update()` on each one. Adding `D1` as a new dependent means `D1` subscribes itself; `A1`'s code doesn't change. This is the same shape behind GUI event listeners, pub/sub message queues, and reactive UI frameworks that re-render a component when a piece of state it reads from changes.

The failure modes are real, though. If `update()` methods trigger further changes on things that are themselves observed, you get a cascade that's genuinely hard to trace by reading any single class. And subscriber lists are a classic memory leak source — an object that goes out of scope without calling `unsubscribe()` stays referenced forever by the subject, invisible until memory profiling turns it up.

## Strategy: swap the algorithm, not the object

Now take a checkout system that needs to apply a discount. The naive version is a method with `if (customerType == "gold") { ... } else if (customerType == "silver") { ... }`, and every new tier means editing that method and re-testing all the branches that already worked. Strategy pulls each branch out into its own class implementing a shared interface — `DiscountStrategy` with one method, `apply(order)` — and the checkout code holds a reference to whichever strategy it was configured with, calling `strategy.apply(order)` without an `if` in sight. Adding a platinum tier means writing `PlatinumDiscountStrategy`; the checkout method that applies the discount is never touched again.

The same shape applies anywhere a family of interchangeable algorithms exists behind one operation — three ways to sort a list, three compression algorithms picked by file type, three ways to validate a password. The context object (the checkout, the sorter) holds a strategy and delegates to it; swapping behavior at runtime is "assign a different strategy instance," not "edit a branch."

Strategy isn't free. It costs an interface and a class per variant, and if an algorithm genuinely only has one implementation and no credible second one on the horizon, that's ceremony with no payoff — a plain method does the same job with less indirection.

## Where the two meet

Observer answers "who finds out when something changes." Strategy answers "which code runs when something needs doing." They're frequently combined: an observer's `update()` method often hands off to a strategy object to decide how to react, rather than hardcoding the reaction inline. Keep them as two separate questions in your head, and both patterns stay simple tools instead of blurring into one vague idea of "flexible code."
