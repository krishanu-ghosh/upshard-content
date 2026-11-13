---
title: "Reactive Programming: Streams Instead of State"
description: Modeling a system as asynchronous streams of events instead of state you poll or mutate — observables, operators, and backpressure.
roadmap: software-architect
node: reactive-programming
group: paradigms
video:
  provider: youtube
  id: "KOjC3RhwKU4"
  start: 0
  end: null
creator:
  name: CS50
  channel_url: https://www.youtube.com/@cs50
duration_seconds: 3496
level: intermediate
prerequisites:
  - functional-programming-concepts
takeaways:
  - "Reactive programming models a program as a set of data streams — sequences of values arriving over time — and declares how those streams transform into other streams, rather than imperatively polling for changes or manually wiring callbacks."
  - "An observable is a source of values over time that a subscriber can react to; it can emit zero or more values and then either complete or error, which is the same three-outcome shape a Promise has, generalized from exactly one value to any number of them."
  - "Operators like map, filter, and debounce transform one stream into another using the same functional-composition habits as map and filter on a list — the stream is just a list whose elements haven't all arrived yet."
  - "Backpressure, at this programming-model level, is the problem of a stream producing values faster than a subscriber can consume them, and operators like throttle, sample, and buffer are how a stream applies a policy for what to do when that happens."
  - "Reactive programming is a genuine fit for UI event handling and real-time data feeds, where state really is a series of events over time, but it adds real complexity and debugging friction that isn't worth paying for code that's just doing one async operation at a time."
check_questions:
  - q: How is an observable similar to, and different from, a Promise?
    a: Both represent an asynchronous result that resolves later and can fail, but a Promise resolves to exactly one value, while an observable can emit zero, one, or many values over time before it completes or errors — it's the stream generalization of a single future value.
  - q: Why is a stream operator like map or filter described as the same idea as list operations, "generalized"?
    a: A list's map/filter transform a fixed, already-available collection; a stream's map/filter apply the exact same transformation logic to values that haven't arrived yet and will keep arriving, so the stream is conceptually a list spread out across time rather than held all at once in memory.
  - q: "Why is reactive programming a poor fit for a simple, one-shot async operation like fetching a user's profile and rendering it?"
    a: A single request/response pairs naturally with an async/await call or a Promise, which any developer reading the code can follow directly; wrapping it in an observable adds subscription management, operator chaining, and a different mental model for no benefit, since there's no ongoing stream of values to react to.
deeper:
  - label: "RxJS Quick Start with Practical Examples"
    url: "https://www.youtube.com/watch?v=2LCo926NFLI"
related_formulas: []
---

## From "ask for a value" to "react to a stream"

Most code is written around a request/response or get/set mental model: call a function, get a value back, move on. Reactive programming starts from a different premise — that a lot of real systems are better modeled as streams of events arriving over time, and your job is to declare how those streams transform into the streams you actually want, rather than to imperatively poll for changes or hand-wire a tangle of callbacks. A button's clicks, a websocket's incoming messages, a sensor's periodic readings, a search box's keystrokes — each of these is naturally a sequence of values spread out over time, not a single value you fetch once. Reactive programming gives that shape a first-class name — a **stream** — and a toolkit for working with it directly instead of reinventing ad-hoc event-handling code every time.

## The observable: a stream you can subscribe to

The central abstraction, in the Reactive Extensions (Rx) family of libraries that popularized this model, is the **observable**. An observable is a source that can emit a sequence of values over time to anyone who subscribes to it, and it eventually either completes (no more values are coming) or errors (something went wrong). This is the same three-outcome shape a `Promise` has — resolve, or reject — generalized from "exactly one eventual value" to "any number of values over any span of time." A `Promise<User>` resolves once with one user; an `Observable<ClickEvent>` can emit an unbounded sequence of click events for as long as the subscription stays open. Framing it this way makes the relationship between the two models concrete: a Promise is the special case of an observable that happens to emit at most one value.

## Operators: the same habits as map and filter, spread across time

Once your data is a stream, you reuse the exact composition habits from functional programming (see `functional-programming-concepts`): `map` transforms each value as it arrives, `filter` lets only matching values through, `debounce` waits for a pause in activity before emitting (useful for a search box that shouldn't fire a request on every keystroke), and `merge` combines multiple streams into one. `clicks.filter(isLeftClick).map(toCoordinates)` reads the same way `list.filter(...).map(...)` does — the difference is that the stream's elements haven't all arrived yet, so the pipeline describes what will happen to each value as it shows up rather than what happens to a collection that's already fully in memory.

## Backpressure, at the programming-model level

When a stream produces values faster than a subscriber can keep up with, something has to give — this is backpressure. System-design's `backpressure-and-flow-control` covers this problem between distributed services (a slow consumer service signaling a fast producer service to slow down); the same problem shows up one level down, inside a single program, when a fast-emitting observable (every mouse-move event, say) outpaces what a subscriber can usefully process. The stream-level tools for this are operators with an explicit policy: `throttle` emits at most one value per time window and drops the rest, `sample` takes a snapshot at fixed intervals, and `buffer` collects values and emits them in batches. Picking among them is picking a policy for what to sacrifice — freshness, completeness, or memory — when the firehose outpaces the sink.

## When this model is worth the complexity

Reactive programming is a strong fit exactly where state really is a sequence of events over time that multiple parts of a system need to react to independently — UI event handling, real-time dashboards, live data feeds, and systems that combine several independent async sources (a websocket, a timer, and user input, say) into one derived result. It's a poor fit for a single, one-shot async call: fetching one record and rendering it is better served by a plain `async`/`await` call that any developer can read top to bottom, and wrapping it in an observable adds subscription lifecycle management (an observable that's never unsubscribed from can leak) and a steeper learning curve for no corresponding benefit. The architectural judgment call, same as with functional programming generally, is recognizing which parts of your system are genuinely stream-shaped and reaching for this model only there — a distinction worth carrying into `the-actor-model` next, which solves a related but different problem: not "how do I compose streams of events," but "how do independent, stateful units communicate safely without shared memory."
