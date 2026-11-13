---
title: "The Prototype Pattern"
description: Cloning an already-configured object instead of building one from scratch — and the shallow-versus-deep-copy trap that makes cloning riskier than it looks.
roadmap: software-design-architecture
node: the-prototype-pattern
group: creational-patterns
video:
  provider: youtube
  id: "DcFhITC9v0E"
  start: 0
  end: null
creator:
  name: Geekific
  channel_url: https://www.youtube.com/@geekific
duration_seconds: 462
level: intermediate
prerequisites: [abstract-factory-and-builder]
takeaways:
  - "Prototype answers a problem the other creational patterns don't: what to do when building an object from scratch is expensive, or when the exact concrete class isn't known to the code that needs a copy of it."
  - "Instead of constructing a new instance, the pattern clones an existing, already-configured one — every type that supports this implements a shared `clone()` method, so cloning code never needs to name the concrete class it's copying."
  - "A naive clone is a shallow copy: it duplicates the object's own fields, but any field that's a reference to a mutable object — a list, another object — ends up shared between the original and the clone, so mutating one through that reference silently mutates the other."
  - "A deep copy recursively clones every referenced mutable object too, so the clone is fully independent of the original; the real design work in Prototype is deciding, field by field, which depth of copy is actually correct."
  - "Prototype beats a factory when construction itself is the expensive part — re-parsing a file, re-running a setup query — because cloning an in-memory object is just copying memory, not redoing the work that built it the first time."
  - "A prototype registry — a lookup of named, pre-configured template instances, cloned on request — lets a system offer new 'kinds' of an object by registering a new template, without ever adding a new class or a new branch to a factory."
check_questions:
  - q: Why can't Factory Method or Abstract Factory help in a situation where the exact concrete class isn't known ahead of time?
    a: Both patterns still have to construct a brand-new instance from scratch, which means some piece of code has to know which concrete class to call `new` on, even if that decision is hidden behind an overridable method; Prototype instead calls `clone()` on an existing object through a shared interface, so the cloning code never needs to reference the concrete class by name at all.
  - q: What's the practical difference between a shallow copy and a deep copy of a configured object?
    a: A shallow copy duplicates the object's own fields but leaves any reference fields pointing at the same underlying mutable objects as the original, so a change made through the clone's reference is visible in the original too; a deep copy also clones every object those references point to, so the clone is a fully independent structure that can be changed without affecting the original.
  - q: When does Prototype actually save work compared to just building a new object?
    a: When constructing the object the normal way is itself expensive — reading a large file, running a database query, performing an expensive computation — because cloning an already-built, in-memory object is just a memory copy, which is far cheaper than repeating whatever work produced that object the first time.
deeper: []
related_formulas: []
---

## When building from scratch is the wrong move

Every creational pattern covered so far — Factory Method, Abstract Factory, Builder — still ends with a fresh call to a constructor somewhere. That's fine as long as constructing the object is cheap and the code doing the constructing knows which concrete class it needs. Two situations break that. Sometimes construction itself is expensive: a `GameLevel` object that gets built by parsing a file, placing two hundred enemies, and computing a collision mesh takes real, measurable time to construct every single time. Other times, the code that needs a new object doesn't actually have the concrete class available to call `new` on — a plugin-supplied shape in a drawing app, for instance, where the application's core code has never seen that class's name. Prototype solves both by skipping construction entirely: instead of building a new object, it clones an existing one.

## Clone instead of construct

The shape is simple: any type that participates implements a `clone()` method, defined once in a shared interface. A drawing app's shape palette holds one configured instance per shape type — `Circle`, `Star`, a user-defined shape — and creating a new shape on the canvas means calling `.clone()` on whichever palette entry the user picked. The app's core code never writes `new Circle()` or checks what type it's cloning; it just calls the one method every prototype guarantees it has. A plugin can register a brand-new shape type the core code has never heard of, and cloning still works, because the contract is "can be cloned," not "is one of these known classes."

The performance case is just as concrete: re-parsing that same level file and re-placing two hundred enemies from scratch costs a few seconds every time a player restarts. Parse it once into a fully-built `GameLevel`, keep that as a template, and every restart becomes `template.clone()` — a few microseconds of copying memory instead of redoing the parse.

## The shallow-copy trap

Cloning isn't automatically correct. A naive `clone()` copies the object's own fields byte for byte — primitives copy cleanly, but a field that's a *reference* to another mutable object, like a `List<Enemy>`, gets copied as a reference too. The clone and the original end up pointing at the exact same list. Move an enemy through the clone, and the "original" level moves it as well, because there was only ever one list underneath both objects. This is a shallow copy, and it's the default behavior of most built-in cloning mechanisms.

A deep copy fixes this by recursively cloning every referenced mutable object, not just the top-level fields, so the clone owns its own independent list, its own independent enemy objects, all the way down. The catch is that deep copying everything by default can be its own waste — if two hundred cloned levels share the exact same, never-mutated texture asset, deep-copying that asset two hundred times burns memory for no benefit. The actual design decision Prototype forces is going field by field and deciding which referenced objects need independent copies and which can safely stay shared, rather than reaching for one blanket copy strategy and hoping it's right everywhere.

## Naming prototypes, and when to skip the pattern

A prototype registry takes this further: a lookup table of named template instances — `"goblin"`, `"orc-captain"` — each cloned on request instead of constructed. Adding a new enemy kind becomes "register a new configured template," often loadable from a data file, rather than writing and compiling a new class. That's a genuine advantage over a factory when kinds multiply quickly and differ mostly in data, not behavior.

None of this is worth reaching for when construction is already cheap and the concrete type is already known at the call site — a plain constructor call does the same job with no `clone()` method to maintain and no shallow-copy bugs to think about. Prototype earns its place specifically where construction is expensive or the concrete type is genuinely unknown to the caller; outside those two situations, it's an extra method and an extra failure mode for a problem that was never actually there.
