---
title: "Adapter and Facade"
description: Adapter translates a mismatched interface so existing code keeps working; Facade hides a tangled subsystem behind one simple entry point.
roadmap: software-design-architecture
node: adapter-and-facade
group: structural-patterns
video:
  provider: youtube
  id: "wA3keqCeKtM"
  start: 0
  end: null
creator:
  name: Geekific
  channel_url: https://www.youtube.com/@geekific
duration_seconds: 300
level: intermediate
prerequisites: [the-prototype-pattern]
takeaways:
  - "Adapter exists because the interface your code needs and the interface you were handed rarely match, and rewriting the thing you were handed usually isn't an option — a third-party SDK's charge(amountCents) method has to satisfy your own PaymentProcessor.pay(amount) interface without either side changing."
  - "An Adapter class implements the interface your code expects and holds the incompatible object inside it, translating each call — pay(amount) becomes charge(amount * 100) — so every other class in the app keeps talking to PaymentProcessor and never learns the vendor's shape exists."
  - "Facade exists for a different reason: not one mismatched interface, but several correct ones that have to be built and called in a specific order — configuring a codec, opening an output stream, setting a bitrate, and starting a muxer, every time, in that sequence."
  - "A Facade is one class with a small method like encode(file) that internally constructs and sequences the subsystem's real classes; callers who need finer control can still reach past the facade and use those classes directly, since a facade narrows the common path without blocking the rest."
  - "Both patterns put one class between your code and someone else's classes, which is why they're easy to blur together — the test is whether the thing behind the wrapper has the wrong interface (Adapter) or the right interface with too many pieces to coordinate by hand (Facade)."
  - "Reach for Adapter when you're integrating against an interface you can't edit; reach for Facade when you own the subsystem but the sequence of calls needed to use it correctly is miserable to repeat at every call site."
check_questions:
  - q: Why can't you just edit the third-party class to match your own interface instead of writing an Adapter?
    a: You usually don't own the third-party code — it's a vendor library or SDK you depend on but can't modify — and even on the rare occasion you could, editing it directly means every future update to that dependency has to be re-patched by hand. An Adapter sits in your own codebase instead, translating calls without touching the vendor's class at all.
  - q: What's the difference between what Adapter hides and what Facade hides?
    a: Adapter hides a single interface mismatch — one method shape that doesn't match what your code expects. Facade hides coordination complexity across several classes that each have a perfectly normal interface on their own; the problem isn't mismatch, it's that using them correctly means calling many of them in a specific order.
  - q: Does putting a Facade in front of a subsystem stop other code from calling the subsystem's classes directly?
    a: No — a Facade is a convenience layer, not a lock. It gives most callers one simple entry point, but any caller that needs finer control is still free to reach past the facade and call the underlying classes directly; nothing about the pattern forbids that.
deeper:
  - label: "The Facade Pattern Explained and Implemented in Java | Structural Design Patterns | Geekific"
    url: "https://www.youtube.com/watch?v=xWk6jvqyhAQ"
related_formulas: []
---

## Fitting into shapes you didn't choose

Most structural patterns solve a version of the same problem: you need to use some code, but the shape it comes in doesn't match the shape you need. Adapter and Facade are the two simplest answers, and they answer different versions of that problem. Adapter is for when exactly one interface is wrong. Facade is for when nothing is technically wrong — there are just too many correct interfaces to deal with at once.

## Adapter: translating one mismatched call into another

Say your app defines a `PaymentProcessor` interface with one method, `pay(amount)`, and every part of your checkout flow is written against it. Now you need a new payment provider, and its SDK ships a `GatewayClient` class with a method called `charge(amountCents)` — cents, not dollars, and a different name entirely. You can't rename the vendor's method; it's their library, and the next SDK update would overwrite your edit anyway.

The fix is a class — call it `GatewayAdapter` — that implements `PaymentProcessor` and holds a `GatewayClient` instance inside it. Its `pay(amount)` method does the one thing the vendor class can't: `charge(amount * 100)`. Every class in your checkout flow keeps calling `pay(amount)` exactly as before; none of them ever learns that `GatewayClient` or its cents-based API exists. Add a second provider later, and it gets its own adapter — the checkout code doesn't change at all.

That's the whole pattern: one class, implementing the interface the caller expects, wrapping the object whose interface doesn't match, translating each call across the gap. The cost is one extra class per incompatible interface you adapt — cheap, against the alternative of rewriting either side.

## Facade: one entry point in front of several correct classes

Facade's starting problem looks different. Nothing is mismatched — a video-encoding subsystem might have a perfectly sensible `CodecConfig`, `OutputStream`, `BitrateController`, and `Muxer`, each with a clean interface of its own. The pain is that encoding a file correctly means constructing all four, in a specific order, with settings that depend on each other, every single time someone wants to encode a file. Every call site that needs encoding either repeats that dance or copies it from somewhere else.

A `VideoEncoderFacade` class fixes this by doing that sequencing once, inside one method: `encode(file)`. Internally it builds the `CodecConfig`, opens the `OutputStream`, wires up the `BitrateController`, and hands everything to the `Muxer` in the right order. Callers who just want a file encoded call `encode(file)` and never see the four classes underneath. Callers who need unusual control — a custom bitrate curve, say — can still reach past the facade and use `CodecConfig` and `Muxer` directly.

## Same shape, different job

Adapter and Facade are easy to blur together because both are "one class standing between your code and someone else's classes." The test is what problem it's solving. If the thing behind the wrapper has the wrong interface, you're adapting. If it has the right interface but too many pieces to call in the right order by hand, you're building a facade. Adapter translates; Facade orchestrates. Getting the two confused doesn't break anything — it's just the wrong word for what you built, and the vocabulary is what lets two engineers agree on a design in one sentence instead of redrawing it on a whiteboard.
