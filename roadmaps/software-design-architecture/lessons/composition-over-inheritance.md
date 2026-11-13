---
title: "Composition Over Inheritance"
description: Why deep inheritance hierarchies get brittle as requirements shift, and the "has-a instead of is-a" fix, worked through the classic Duck example.
roadmap: software-design-architecture
node: composition-over-inheritance
group: design-principles
video:
  provider: youtube
  id: "hxGOiiR9ZKg"
  start: 0
  end: null
creator:
  name: CodeAesthetic
  channel_url: https://www.youtube.com/@CodeAesthetic
duration_seconds: 600
level: beginner
prerequisites: [separation-of-concerns-and-coupling-and-cohesion]
takeaways:
  - "The fragile base class problem is that any change to a shared base class risks breaking every subclass that inherits from it, even subclasses whose authors never touched the base class and don't know its internals changed."
  - "A deep inheritance hierarchy encodes assumptions about which behaviors travel together — the moment a new requirement needs one behavior without another, the hierarchy has to be reshuffled or duplicated around."
  - "Composition replaces \"is a kind of\" with \"has a\" — a `Duck` doesn't inherit flying behavior, it holds a reference to a `FlyBehavior` object and delegates to it."
  - "Composition lets behavior change at runtime, not just at compile time — swapping a `Duck`'s `FlyBehavior` object changes how it flies without touching the `Duck` class or any other duck."
  - "Inheritance is still the right tool when the subtype genuinely never needs to vary independently of the base type and the hierarchy is shallow and stable — the fix isn't banning inheritance, it's not reaching for it as the default."
  - "The same choice shows up under different names across the codebase — strategy objects, dependency injection, plugin systems — all of them are composition over inheritance applied to a specific problem."
check_questions:
  - q: What exactly is the "fragile base class problem"?
    a: It's the risk that changing a base class — even a change that looks safe in isolation, like adjusting what a method does internally — can silently break any subclass that depended on the old behavior, because subclasses are coupled to the base class's implementation details, not just its documented interface.
  - q: Why does a deep inheritance hierarchy get harder to extend as requirements change, rather than easier?
    a: Each level of the hierarchy bundles a specific combination of behaviors together, so adding a new combination that wasn't anticipated — a flying, non-quacking duck, say — often forces either duplicating code across branches of the hierarchy or awkwardly overriding a method to do nothing, both of which are signs the hierarchy no longer matches the real variation in the domain.
  - q: In the classic Duck example, what specifically moves from inheritance to composition?
    a: Flying and quacking move out of the `Duck` class and its subclasses and become their own interfaces — `FlyBehavior` and `QuackBehavior` — with separate implementing classes for each variant; a `Duck` holds one object of each interface and delegates `fly()` and `quack()` calls to them, so a new duck type is assembled by picking existing behavior objects instead of writing a new subclass.
deeper:
  - label: "Strategy Pattern – Design Patterns (ep 1)"
    url: "https://www.youtube.com/watch?v=v9ejT8FO-7I"
related_formulas: []
---

## Inheritance models "is a," but requirements don't stay still

Inheritance is a natural first tool for sharing behavior: a `Duck` is a `Bird`, so `Duck` extends `Bird` and gets `fly()` for free. The model works as long as every subtype really does share the base type's behavior in exactly the way the hierarchy assumes. The trouble starts when a requirement shows up that the original hierarchy never anticipated — a `RubberDuck` that is still very much a `Duck` in every business sense, except it can't fly and doesn't quack. Now `RubberDuck` has to override `fly()` to do nothing, or throw an exception, which means the class is lying about what it can do just to satisfy a hierarchy built around an assumption that no longer holds for every member.

## The fragile base class problem

Deep hierarchies compound this in a second way: the fragile base class problem. A subclass doesn't just inherit a base class's public interface — it inherits, and often quietly depends on, assumptions about how the base class's methods are implemented internally. Change how `Bird.move()` calculates its internal state, intending it as a safe, contained edit, and every subclass that happened to rely on the old internal behavior — maybe one overrode a related method expecting a particular call order — breaks, even though nobody touched that subclass's code. The deeper and more widely-used the base class, the more subclasses are exposed to any change made to it, and the more nervous any edit to it has to make you.

## From "is a" to "has a"

The fix is to stop asking "what is this thing" and start asking "what can this thing do, and where does that capability come from." Instead of `Duck` inheriting `fly()` from `Bird`, give `Duck` a field of type `FlyBehavior` — an interface with one method, `fly()` — and have `Duck.fly()` just call `flyBehavior.fly()`. Write `FlyWithWings` and `FlyNoWay` as two separate classes implementing `FlyBehavior`. A `RubberDuck` is constructed with a `FlyNoWay` behavior object; a `MallardDuck` is constructed with a `FlyWithWings` one. Neither class overrides anything to fake a capability it doesn't have — each just holds the behavior object that matches what it can actually do. `QuackBehavior` works the same way, independently, which means flying and quacking can now vary separately instead of being locked together by a shared ancestor.

## What this buys you that inheritance can't

Because each duck holds a behavior object rather than inheriting a method body, that object can be swapped at runtime — a `Duck` that gets a jetpack mid-flight can have its `FlyBehavior` replaced with a `FlyWithRocket` instance without touching the `Duck` class, the `Bird` hierarchy, or any other duck in the system. Try that with inheritance and there's no clean answer: an object can't change its own class at runtime. Composition also means a new combination of behaviors — a duck that flies with wings but doesn't quack at all — is built by picking two existing behavior objects, not by adding a new branch to a hierarchy or duplicating code across two existing branches.

## Where inheritance still earns its place

None of this makes inheritance wrong by default. A shallow hierarchy where every subtype genuinely, permanently shares the base type's behavior — and where that sharing is extremely unlikely to need to vary per-instance or change at runtime — is simpler to read as inheritance than as composition wired together through constructor parameters. The judgment call is the same one separation of concerns and coupling raise elsewhere: is this relationship stable and exclusive enough that a rigid hierarchy is actually the simplest honest model, or is it a guess about today's requirements calcified into a structure that tomorrow's requirements will have to fight against. Composition costs a bit more wiring up front; inheritance costs more the day a subtype needs to be only partly like its parent.
