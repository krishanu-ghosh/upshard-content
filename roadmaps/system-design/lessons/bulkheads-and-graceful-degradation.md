---
title: "Bulkheads and Graceful Degradation"
description: Isolating resource pools so one failing dependency can't exhaust everything, plus feature flags and fallbacks for degrading gracefully under partial failure.
roadmap: system-design
node: bulkheads-and-graceful-degradation
group: reliability-and-resilience-patterns
video:
  provider: youtube
  id: "d-QQvJIBrUc"
  start: 0
  end: null
creator:
  name: CodeOpinion
  channel_url: https://www.youtube.com/@CodeOpinion
duration_seconds: 483
level: intermediate
prerequisites:
  - circuit-breakers-and-retries
takeaways:
  - A bulkhead partitions a shared resource — threads, connections, CPU — into separate pools per dependency, so one slow dependency can only exhaust its own pool, not the whole service.
  - Without bulkheads, a single misbehaving downstream call can tie up every worker thread in a service, starving requests that have nothing to do with the failing dependency.
  - Bulkheads can be implemented with separate thread pools, separate connection pools, or just semaphores that cap concurrent calls to a given dependency.
  - Graceful degradation means a service keeps serving its core function with reduced quality when a dependency fails, instead of failing the whole request.
  - A feature flag that disables a non-critical dependency on demand turns "degrade under failure" from an emergency code change into a switch someone can flip during an incident.
  - Fallback responses — cached data, a default value, or simply omitting a non-essential section of a response — are what graceful degradation looks like at the API layer.
check_questions:
  - q: A service uses one shared thread pool for calls to both its payments API and its recommendations API. The recommendations API starts timing out. What happens, and how would a bulkhead prevent it?
    a: Every thread that calls the slow recommendations API gets stuck waiting, and since the pool is shared, those stuck threads eventually consume all available workers — so payment requests start failing too, even though payments itself is healthy; a bulkhead gives recommendations its own separate, smaller thread pool so its slowness can only exhaust that pool, leaving threads free for payments.
  - q: What's the difference between a circuit breaker and a bulkhead, given they're both used for the same dependency?
    a: A circuit breaker decides whether to call the dependency at all based on its recent failure rate, while a bulkhead limits how many concurrent calls to that dependency are allowed regardless of whether they're succeeding — they solve different problems and are normally used together, not as alternatives.
  - q: An e-commerce product page shows price, stock, and "customers also bought" recommendations. The recommendations service is down. What does graceful degradation look like here, concretely?
    a: The page renders normally with price and stock, simply omitting the recommendations section or replacing it with a cached/generic list, instead of the entire product page failing to load because one non-essential dependency is unavailable.
---

## Why isolation has to happen before failure, not after

A circuit breaker and a sensible retry policy assume you already know a call is failing and are deciding what to do about it. Bulkheads and graceful degradation are about structuring the system *before* that happens, so that when one piece does fail, the blast radius is contained to that piece instead of spreading to everything else that happens to share its resources.

## The bulkhead pattern

The name comes from ship design: a ship's hull is divided into watertight compartments so a single breach floods one compartment instead of sinking the whole vessel. Applied to software, a bulkhead means partitioning a shared resource — most commonly thread pools, connection pools, or just a concurrency limit — per dependency, so that one dependency behaving badly can only consume the resources allocated to it.

The classic failure this prevents: a service calls three downstream APIs (say, inventory, pricing, and recommendations) using one shared pool of 100 worker threads. Recommendations starts responding slowly — not failing outright, just slow, which is often worse than an outright failure because nothing trips a timeout fast. Every request that touches recommendations now holds a thread for much longer than normal. Within a few seconds, all 100 threads are tied up waiting on the slow recommendations calls, and requests that only need inventory or pricing — which are both perfectly healthy — can't get a thread to run on at all. A dependency that's merely "a bit slow" has taken down the entire service.

A bulkhead fixes this by giving recommendations its own pool — say, 10 threads, sized to what it can afford to lose. When it degrades, at most those 10 threads get stuck; the other 90 keep serving inventory and pricing requests without interruption. The same idea applies to connection pools (don't let one database's connections starve another's) and to simple semaphores that just cap "no more than N concurrent calls to this dependency" without a dedicated thread pool at all — the lightest-weight form of the pattern.

Sizing bulkheads is a real design decision, not an afterthought: too generous and the isolation doesn't actually bound the damage; too stingy and you throttle a dependency that's healthy and simply popular. The sizing question is usually "how many concurrent calls to this dependency can the rest of the system tolerate losing, worst case?"

## Graceful degradation: keep serving, just serve less

Bulkheads contain the damage; graceful degradation is the decision about what to do with a request once one of its dependencies is unavailable. The alternative — fail the entire request because one non-essential piece of it failed — is usually the wrong default, but it's the default you get unless someone deliberately designs against it.

Take a product page that needs price, stock level, and a "customers also bought" recommendation strip. Price and stock are essential — without them, the page is useless. Recommendations are not — a product page without suggestions is a worse page, not a broken one. If the recommendations service is down, the degraded response is: render the page normally, and simply leave out (or replace with something generic/cached) the recommendations section. The user gets a slightly less personalized but fully functional page instead of a spinning loader or an error screen.

This requires treating "what happens if X is unavailable" as a first-class part of designing the call to X, not an exception handler bolted on later. Concretely, that usually means:

- **A static or cached fallback value** — serve yesterday's cached recommendations, or a generic "trending now" list, instead of a personalized one that requires the live service.
- **A default that's safe even if wrong** — if a feature-eligibility check times out, defaulting to "not eligible" is usually safer than defaulting to "eligible," because the cost of the two mistakes isn't symmetric.
- **Omission** — the section simply doesn't render, and the rest of the page is unaffected, which is often the simplest and best option for genuinely non-essential content.

## Feature flags as an operational degradation switch

Writing the fallback logic is half the work; the other half is being able to *turn a dependency off* quickly during an actual incident, without a deploy. A feature flag that gates a non-critical dependency — "render recommendations only if `recommendations_enabled` is true" — turns graceful degradation from something that only kicks in automatically when the dependency times out into something an on-call engineer can also trigger proactively. If recommendations is healthy but suspiciously slow and you suspect it's about to fall over, flipping the flag off *before* it fails protects the rest of the page without waiting for a bulkhead or circuit breaker to react on their own. This is also exactly how incidents get resolved fast in practice: not by shipping a code fix under pressure, but by flipping a switch that was built in ahead of time for precisely this moment.

Bulkheads, fallbacks, and feature flags aren't three unrelated tools — they're the same underlying idea applied at three different layers: contain the resource damage, decide what a degraded response looks like, and give a human a lever to invoke that decision on demand.
