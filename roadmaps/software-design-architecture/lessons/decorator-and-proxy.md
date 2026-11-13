---
title: "Decorator and Proxy"
description: Decorator and Proxy both wrap an object behind its own interface — Decorator adds behavior at runtime, Proxy controls access to the real thing.
roadmap: software-design-architecture
node: decorator-and-proxy
group: structural-patterns
video:
  provider: youtube
  id: "v6tpISNjHf8"
  start: 0
  end: null
creator:
  name: Geekific
  channel_url: https://www.youtube.com/@geekific
duration_seconds: 401
level: intermediate
prerequisites: [adapter-and-facade]
takeaways:
  - "Decorator and Proxy both wrap an object behind the identical interface the wrapped object implements, which is exactly why their code looks alike and their intents get confused — the difference is what the wrapper is for, not its shape."
  - "Decorator exists to add behavior to an object at runtime without subclassing it — a plain FileStream can be wrapped in a LoggingStream, then a CompressingStream, each implementing the same interface and delegating to the one it wraps before adding its own work."
  - "Subclassing every combination of optional behaviors grows combinatorially — five independent behaviors would mean thirty-two subclasses — while Decorator needs one class per behavior, stacked in whatever order a given object needs it."
  - "Proxy exists to control whether, when, or how a call to the real object happens — a LazyImageProxy implements the same interface as a real HighResImage but doesn't load the file from disk until render() is actually called for the first time."
  - "A Proxy can also sit in front of the real object for access control (checking permissions before forwarding a call) or remote access (turning a local-looking call into a network request), which is why lazy loading, access control, and remote stubs are all the same pattern underneath."
  - "Stacking several Decorators on one object is normal and expected; stacking several Proxies rarely makes sense, because a Decorator adds an independent, composable behavior while a Proxy is one gate in front of the real object, and a second gate duplicates the first rather than adding anything new."
check_questions:
  - q: Why can a Decorator and a Proxy look nearly identical in code even though they solve different problems?
    a: Both are a class implementing the same interface as the object they wrap, holding a reference to that object, and forwarding calls to it — that structure is identical either way. What distinguishes them is intent — whether the wrapper is adding behavior the caller wants, or deciding whether and how to let a call through to the real object.
  - q: Why does it make sense to stack multiple Decorators but not multiple Proxies?
    a: Each Decorator adds its own independent behavior — logging, then compression, then both — and those behaviors compose cleanly in any order. A Proxy is a single gate controlling access to the real object; stacking a second proxy in front of the first doesn't add a new behavior, it just duplicates the gate the first one already provides.
  - q: How does a lazy-loading image proxy save work without the caller knowing anything changed?
    a: The proxy implements the exact same interface as the real image class, so callers call render() on it exactly as they would on the real object. Internally, the proxy doesn't load the (possibly large) file from disk until render() is called for the first time — every call before that is free, and the caller never has to check whether the image is loaded yet.
deeper:
  - label: "The Proxy Pattern Explained and Implemented in Java | Structural Design Patterns | Geekific"
    url: "https://www.youtube.com/watch?v=TS5i-uPXLs8"
related_formulas: []
---

## Same wrapper shape, two different jobs

Decorator and Proxy look nearly identical in code — both wrap an object behind the same interface that object already implements — which is exactly why they're taught side by side. What splits them apart isn't structure, it's intent: one pattern adds behavior the caller wants, the other controls access to behavior the caller already has a right to.

## Decorator: stacking behavior without a subclass explosion

Take a plain `FileStream` with a `write(bytes)` method. Now you want logging on some streams, compression on others, and both on a few. Writing `LoggingStream`, `CompressingStream`, `LoggingCompressingStream`, and `CompressingLoggingStream` as four separate subclasses doesn't scale — five optional behaviors would mean thirty-two classes to cover every combination.

Decorator fixes this by making each behavior its own class that implements the same interface as the thing it wraps and holds a reference to it. `LoggingStream` implements `FileStream`, takes a `FileStream` in its constructor, and its `write(bytes)` method logs the call and then delegates to the wrapped stream's `write(bytes)`. `CompressingStream` does the same thing with compression instead of logging. Stack them — `new LoggingStream(new CompressingStream(plainStream))` — and both behaviors apply in whatever order you wrapped them, built from two classes instead of four. The combination is decided where the object is constructed, not baked into a subclass name at compile time.

## Proxy: same interface, different intent

A Proxy wraps an object behind the identical interface too, but it isn't adding anything the caller asked for — it's standing in for the real object and deciding whether, when, or how to let a call through. A `LazyImageProxy` implements the same `Image` interface as a real `HighResImage`, but it doesn't load the multi-megabyte file from disk until `render()` is actually called for the first time; every other call bounces off the proxy for free until then. An `AccessControlProxy` implements the same interface as a `DocumentEditor` but checks the caller's permissions before forwarding `save()` to the real editor. A remote proxy implements the same interface as a service running on another machine and turns what looks like a local call into a network request behind the scenes.

## The question that tells them apart

Read purely as code, a Decorator and a Proxy can be identical: a class implementing an interface, holding an instance of that same interface, forwarding calls to it. The question that separates them is what the wrapper is for. If it's adding behavior the caller wants and expects — logging, compression, formatting — it's a Decorator, and you can usually stack several. If it's controlling whether, when, or how the real call happens — lazy loading, a permission check, hiding a network hop — it's a Proxy, and there's typically one gate to pass through, not several optional layers to add.

## Picking between them

Reach for Decorator when the real object already works correctly and you want to add something around every call without touching its class. Reach for Proxy when the real object is expensive to create, sensitive to access, or physically somewhere else, and the point of the wrapper is to manage that fact — not to add a feature, but to protect or defer one.
