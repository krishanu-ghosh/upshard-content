---
title: "Composite, Bridge, and Flyweight"
description: Composite unifies a tree under one interface, Bridge splits two axes of variation apart, and Flyweight shares identical state across millions of objects.
roadmap: software-design-architecture
node: composite-bridge-and-flyweight
group: structural-patterns
video:
  provider: youtube
  id: "oo9AsGqnisk"
  start: 0
  end: null
creator:
  name: Geekific
  channel_url: https://www.youtube.com/@geekific
duration_seconds: 336
level: intermediate
prerequisites: [decorator-and-proxy]
takeaways:
  - "Composite lets calling code treat a single object and a group of objects through the same interface — a File and a Folder both implement a Node interface with one method, size(), so a Folder.size() recurses over its children without ever needing to check whether a given child is a File or another Folder."
  - "Without Composite, any code that walks a tree of mixed node types has to branch on type before calling anything — exactly the kind of type-check logic the Strategy and Open/Closed patterns elsewhere in this roadmap exist to remove."
  - "Bridge exists for a class that varies along two independent dimensions at once — a Shape with multiple kinds and multiple renderers — where subclassing both dimensions together multiplies the subclass count every time either dimension grows."
  - "Bridge splits the two dimensions into separate hierarchies connected by composition instead of inheritance: Shape holds a Renderer interface rather than extending one, so Circle and Square extend Shape while RasterRenderer, VectorRenderer, and WireframeRenderer implement Renderer on their own, and adding a fourth renderer means writing one new class, not one per shape."
  - "Flyweight exists for programs holding huge numbers of near-identical objects, like a text editor rendering a million characters — it separates what's shared across instances (a glyph's font, weight, and outline) from what varies per instance (its position on the page), caching one shared Glyph object per distinct combination instead of creating a million independent copies."
  - "All three patterns trade a naive, combinatorial design for one that scales differently: Composite turns type-checking into one shared method call, Bridge turns multiplied subclasses into two independent hierarchies, and Flyweight turns one object per instance into one shared object per distinct value."
check_questions:
  - q: Why doesn't code calling size() on a filesystem tree need to check whether it's looking at a File or a Folder?
    a: Because both implement the same Node interface and both provide a size() method — a File returns its own byte count, and a Folder sums its children's size() calls, each of which is itself a Node and could be either kind. The caller only ever calls node.size(); which concrete type is underneath is irrelevant to it.
  - q: What problem does Bridge solve that subclassing both dimensions together creates?
    a: When a class varies along two independent dimensions — say, shape kind and rendering style — subclassing both together produces one subclass per combination, so the subclass count multiplies every time either dimension grows. Bridge keeps the two dimensions in separate hierarchies connected by a held reference, so adding a new value on either side means adding one new class, not one per combination.
  - q: How does Flyweight cut memory use without losing each character's individual position on the page?
    a: It splits a glyph's data into what's identical across every instance of that character (font, weight, outline) and what's unique to each instance (its x/y position). The shared data lives in one cached Glyph object per distinct combination; the position lives in a small, separate record for each character on the page, so a million characters need a million lightweight positions but only a few hundred shared glyphs.
deeper:
  - label: "The Bridge Pattern Explained and Implemented in Java | Structural Design Patterns | Geekific"
    url: "https://www.youtube.com/watch?v=88kAIisOiYs"
  - label: "The Flyweight Pattern Explained and Implemented in Java | Structural Design Patterns | Geekific"
    url: "https://www.youtube.com/watch?v=qscOsQV-K14"
related_formulas: []
---

## Three ways to handle "too many" without too much code

Composite, Bridge, and Flyweight all show up once a program has a large number of similar objects, and each solves a different version of "too many." Composite handles a tree of them that calling code wants to treat as one thing. Bridge handles two independent dimensions of variation that would otherwise multiply into a wall of subclasses. Flyweight handles enormous counts of small objects whose memory footprint would sink the program if each one carried its own copy of data that's identical across all of them.

## Composite: treating one object and a group the same way

A filesystem is a tree of `File` and `Folder` objects, and most code that walks it wants to ask the same question regardless of which kind it's looking at: how big is this? A `File.size()` just returns its byte count. A naive `Folder.size()` needs a loop summing its children, and any caller would need to check "is this a file or a folder?" before deciding which method to call — exactly the type-check branching the rest of this roadmap has been trying to get rid of.

Composite's fix: both `File` and `Folder` implement the same `Node` interface with one method, `size()`. `File.size()` returns its own byte count. `Folder.size()` loops over its children and sums each child's `size()` — and since every child is also a `Node`, that recursion doesn't care whether a given child is a file or another folder. Calling code never branches on type; it calls `node.size()` on the root, and the tree sums itself.

## Bridge: letting two kinds of variation move independently

A `Shape` with subclasses `Circle` and `Square` is fine until a second axis of variation shows up — say, three ways to render a shape: raster, vector, and wireframe. Subclassing both axes together produces `RasterCircle`, `VectorCircle`, `WireframeCircle`, `RasterSquare`, and so on — the shape count times the renderer count, and every new renderer or shape multiplies the rest.

Bridge splits the two axes into separate hierarchies connected by a reference, not inheritance. `Shape` holds a `Renderer` interface instead of extending one; `Circle` and `Square` extend `Shape`; `RasterRenderer`, `VectorRenderer`, and `WireframeRenderer` implement `Renderer` independently. A `Circle` built with a `VectorRenderer` calls `renderer.drawArc(...)` instead of implementing its own vector-drawing code. Adding a fourth renderer means writing one new `Renderer` class, not one new subclass per shape.

## Flyweight: sharing what's identical, storing only what differs

A text editor rendering a million characters needs some representation per character — font, weight, size, the rendered outline — but a million full copies of that data is a memory problem, since every lowercase "e" in the same font and size is identical to every other. Flyweight separates what's shared (the glyph's visual data) from what's per-instance (its position on the page). One `Glyph` object per distinct character/font/size combination gets created and cached; a `GlyphFactory` hands out the existing one whenever the same combination is requested again, instead of building a new one. A million characters on the page become a million lightweight position records pointing at a few hundred shared `Glyph` objects, not a million heavyweight glyphs.

## What the three have in common

Each pattern trades a naive, usually combinatorial design for one that scales: Composite turns "treat these differently by type" into "call the same method on all of them." Bridge turns "one subclass per combination" into "two independent hierarchies connected by composition." Flyweight turns "one object per instance" into "one shared object per distinct value, referenced by many instances." None of the three is free — Composite can make leaf-only operations awkward to express, Bridge adds a layer of indirection even when only one axis will ever actually vary, and Flyweight only pays off when the shared data genuinely dwarfs the per-instance data. Recognizing which of these three shapes a problem is taking is what tells you which pattern you've just run into.
