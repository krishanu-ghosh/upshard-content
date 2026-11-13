---
title: "Iterator, Mediator, Memento, and Visitor"
description: "Four more behavioral patterns: collection traversal, centralized object communication, state snapshots for undo, and adding operations without touching classes."
roadmap: software-design-architecture
node: iterator-mediator-memento-and-visitor
group: behavioral-patterns
video:
  provider: youtube
  id: "UQP5XqMqtqQ"
  start: 0
  end: null
creator:
  name: Geekific
  channel_url: https://www.youtube.com/@geekific
duration_seconds: 346
level: intermediate
prerequisites: [template-method-and-chain-of-responsibility]
takeaways:
  - "Iterator gives a collection a `hasNext()`/`next()` interface so code can walk through its elements one at a time without ever seeing whether the collection is a tree, a linked list, or an array underneath."
  - "Mediator centralizes many-to-many communication through one coordinator object, so a chat room's members talk to the room, never directly to each other — adding a tenth member means one new connection to the mediator, not nine new connections to everyone else."
  - "Memento captures an object's internal state in a separate snapshot object, then hands that snapshot back later to restore the original state, without ever exposing the object's private fields to whatever is storing the snapshot."
  - "Visitor lets you add a new operation across a whole family of classes by writing one new `Visitor` implementation, using double dispatch so the right method runs for each concrete type without a single `instanceof` check."
  - "All four patterns trade a small amount of extra structure — an iterator object, a mediator class, a memento object, a visitor interface — for avoiding a worse failure: exposed internals, tangled object graphs, broken encapsulation, or classes edited every time a new operation is needed."
check_questions:
  - q: What does Iterator let calling code do that it couldn't do safely otherwise?
    a: It lets code loop over a collection's elements using only `hasNext()` and `next()`, without knowing or depending on whether the collection is an array, a linked list, or a tree — so the collection's internal structure can change later without breaking every loop written against it.
  - q: Why does adding a new member to a chat room built with Mediator only require one new connection instead of several?
    a: Every member only ever talks to the mediator, not to each other directly, so a new member connects once to the central coordinator and can immediately reach everyone already connected to it — without Mediator, each new member would need a direct reference to every existing member, growing connections quadratically.
  - q: How does Visitor add a new operation to a family of classes without modifying any of them, and what does "double dispatch" mean here?
    a: You write one new class implementing the `Visitor` interface with a `visit()` method per concrete type, and each existing class gets (once) a single `accept(visitor)` method that calls `visitor.visit(this)` — double dispatch means the actual method chosen depends on both the visitor's type and the element's type, which is why a new operation doesn't require touching the element classes again after that one-time `accept()` addition.
deeper:
  - label: "The Mediator Pattern Explained and Implemented in Java | Geekific"
    url: "https://www.youtube.com/watch?v=35D5cBosD4c"
  - label: "The Memento Pattern Explained and Implemented in Java | Geekific"
    url: "https://www.youtube.com/watch?v=_Q5rXfGuyLQ"
related_formulas: []
---

## Four narrower problems, four narrower fixes

These four patterns show up less often than Observer or Strategy, but each solves a specific, recurring annoyance precisely enough that reinventing it badly is the more common mistake. None needs the same depth as the earlier lessons — once you see the one problem each one targets, the rest is detail.

## Iterator: don't expose what the collection looks like inside

Looping over an array with an index works fine until the "collection" is a tree, a hash map, or something assembled from two other collections stitched together — code that walked the array by index now has nowhere to put that index. Iterator fixes this by giving every collection a traversal object with `hasNext()` and `next()`, so a `for` loop calls only those two methods and never touches the collection's internal representation. A tree's iterator can do a depth-first walk internally while the calling code sees a flat sequence. Skip this when the structure is genuinely just an array you're never going to change — a raw index loop is simpler and the abstraction buys nothing.

## Mediator: stop objects from wiring themselves directly to each other

A chat room where every member holds a direct reference to every other member works for three people and becomes unmanageable at thirty — each new member means wiring up connections to everyone already there. Mediator puts one coordinator object in the middle: members send messages to the room, and the room decides who receives them. An air-traffic control tower is the same shape — planes talk to the tower, never directly to each other, so adding a plane means one new connection, not one per plane already in the air. The cost is a central object that now knows about everyone, which can become its own tangle if it starts absorbing logic that belongs elsewhere.

## Memento: snapshot state without exposing it

Undo needs some way to go back to a previous state, but letting outside code directly read and later overwrite an object's private fields breaks encapsulation — now everything that ever touches those fields has to understand the object's internals. Memento solves this by having the object itself produce an opaque snapshot object on request, and later accept that exact snapshot back to restore itself, without the thing storing the snapshot (an undo manager, say) ever looking inside it. It's a narrower, state-focused cousin of Command's `undo()` — Memento is specifically about *storing* the state to revert to, not about the action that caused the change.

## Visitor: add an operation without editing the classes it operates on

Say you have `Circle`, `Square`, and `Triangle` classes, and you need to add an `exportToSvg()` operation across all three. Adding a method to each class works until the fifth new operation shows up and now every shape class is cluttered with unrelated export logic. Visitor adds one `accept(visitor)` method to each shape (once), and each new operation becomes a new `Visitor` implementation with a `visit(Circle)`, `visit(Square)`, `visit(Triangle)` method apiece — double dispatch picks the right one automatically. The trade: every new *shape* now means updating every existing visitor, so Visitor fits best when the set of classes is stable and the set of operations keeps growing — not the other way around.

## Less common, not less useful

None of these four come up as often as Observer or Strategy, which is exactly why they're covered together here rather than each getting its own lesson. Recognize the shape of the problem — exposed traversal, tangled object references, broken encapsulation on undo, or classes edited for every new operation — and the right pattern is usually obvious once you've seen it named once.
