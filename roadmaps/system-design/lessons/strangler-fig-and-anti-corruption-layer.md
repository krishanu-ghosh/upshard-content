---
title: "Strangler Fig and Anti-Corruption Layer"
description: Migrating a legacy monolith incrementally by routing growing slices of traffic to new services, and shielding new code from the old system's data model.
roadmap: system-design
node: strangler-fig-and-anti-corruption-layer
group: architecture-patterns-at-scale
video:
  provider: youtube
  id: "m0yuZCAIYqU"
  start: 0
  end: null
creator:
  name: AWS Events
  channel_url: https://www.youtube.com/@AWSEventsChannel
duration_seconds: 1414
level: intermediate
prerequisites:
  - monolith-vs-microservices
takeaways:
  - The strangler fig pattern migrates a legacy system by routing a growing slice of traffic to new services while the legacy system keeps handling everything else, instead of attempting a big-bang rewrite.
  - A routing facade (often a reverse proxy or API gateway) decides, per request, whether the legacy system or a new service handles it — that routing layer is the whole mechanism.
  - The pattern is named for the strangler fig tree, which grows around a host tree and gradually takes over its structural role until the host can be removed.
  - An anti-corruption layer (ACL) is a translation boundary that converts between a legacy system's awkward data model and a new service's clean domain model, so the new code never has to speak the legacy system's language directly.
  - Big-bang rewrites fail more often than incremental migrations because they freeze both old and new systems from shipping real functionality for the whole rewrite duration, while business requirements keep changing underneath them.
  - Data is usually the hardest part of a strangler migration, not code — you need a plan for which system owns the source of truth for a given entity while both systems are live.
check_questions:
  - q: What specifically does the strangler fig pattern avoid, and what does it do instead?
    a: It avoids a big-bang rewrite — replacing the whole legacy system at once — and instead routes a growing slice of traffic, one capability or domain boundary at a time, to new services while the legacy system continues serving everything not yet migrated.
  - q: What is the single mechanism that makes a strangler fig migration work?
    a: A routing facade in front of the legacy system that inspects each incoming request and decides whether to forward it to the legacy system or to the new service that has taken over that capability.
  - q: Why would a new service want an anti-corruption layer in front of a legacy system it still depends on, instead of calling the legacy system's API directly?
    a: Because the legacy system's data model is often inconsistent, poorly named, or shaped by years of workarounds — calling it directly would force that mess into the new service's domain model, so the ACL translates at the boundary and keeps the new code's model clean.
related_formulas: []
---

## Why not just rewrite it?

A full rewrite of a legacy system is tempting and usually fails for the same reason: it freezes delivery. While the rewrite is in progress, the legacy system still has to be maintained and still has to absorb new business requirements, so the rewrite is chasing a moving target. Meanwhile the new system delivers zero value until the (often optimistic) day it's declared complete and cut over wholesale — a single high-stakes event where any gap between the old and new system's behavior becomes a production incident. Many of these projects are abandoned partway through, leaving a half-finished rewrite and an un-maintained legacy system both in production.

## The strangler fig pattern

Martin Fowler named this pattern after the strangler fig, a plant that germinates in the canopy of a host tree, sends roots down around the host's trunk, and over years gradually takes over the host's structural role — until the original tree can die and rot away, leaving the fig standing in its place. Applied to software: instead of rewriting everything before shipping anything, you build new services for one capability at a time, put a routing layer in front of the legacy system, and redirect just that capability's traffic to the new service. The legacy system keeps running, untouched, for everything that hasn't been migrated yet.

Concretely, this usually looks like a reverse proxy or API gateway that sits in front of the legacy monolith and makes a per-request routing decision — most often based on the URL path or a feature flag:

```
Client → Routing Facade
            ├── /api/orders/*      → New Orders Service
            ├── /api/inventory/*   → New Inventory Service
            └── everything else    → Legacy Monolith
```

Each migrated capability ships independently, gets tested in production with real traffic at low risk (you can roll a route back to the legacy system instantly if something's wrong), and delivers value immediately instead of waiting for a big cutover. Over time, the routing facade sends a shrinking slice of traffic to the legacy system, until eventually nothing is left for it to handle and it can be decommissioned — the fig has fully replaced the host tree.

The hard part is almost never the routing logic — it's data. If `Orders` is being migrated but `Inventory` (still legacy) needs to read order data, you need an explicit decision about which system owns the authoritative copy of orders during the transition, and how the other system gets a consistent view of it: dual writes, a change-data-capture pipeline (see `change-data-capture-for-data-pipelines`), or a temporary API call back into whichever system currently owns that data. Skipping this step is the most common way strangler migrations end up with subtly inconsistent data between old and new.

## The anti-corruption layer

A legacy system's internal data model is rarely clean — years of schema patches, overloaded columns, and business logic encoded as magic values accumulate. If a new service calls the legacy system's API directly and maps its responses straight into the new service's domain objects, that mess leaks straight into the new code. Six months later, the new "clean" service has inherited the old system's confusing column names and implicit assumptions anyway.

An **anti-corruption layer (ACL)** is a thin translation boundary placed between the new service and the legacy system specifically to prevent that. The new service only ever talks to the ACL, using its own clean domain language; the ACL is the only piece of code that understands the legacy system's awkward shapes and translates them in both directions.

```
New Order Service  ←→  Anti-Corruption Layer  ←→  Legacy Monolith
  (clean domain:          (translates:              (messy reality:
   Order, LineItem)        legacy "ORD_STAT"          status codes,
                            code → OrderStatus         nullable flags,
                            enum, etc.)                 implicit rules)
```

This is deliberately similar in shape to the adapter pattern from object-oriented design, but applied at a system boundary rather than a class boundary, and usually carrying more translation logic — unit conversions, status code mapping, filling in defaults the legacy system leaves implicit, or reconciling a field that means two different things in the two systems. When the legacy system is eventually retired, you delete the ACL along with it; none of that translation logic has leaked into the new service's core domain model, which is the whole point.

## Using them together

In a strangler migration, the routing facade decides *which* system handles a request, and the anti-corruption layer governs *how* the new system talks to the part of the legacy system it still depends on. A new service migrated early in the process will likely need an ACL to keep calling the still-legacy parts of the system cleanly; as more of the system migrates, the ACL's surface area shrinks along with the legacy system behind it, until both disappear together. Used this way, the two patterns turn a risky, all-or-nothing rewrite into a long series of small, reversible, individually low-risk steps — which is the entire reason this approach tends to actually finish, where big-bang rewrites tend not to.
