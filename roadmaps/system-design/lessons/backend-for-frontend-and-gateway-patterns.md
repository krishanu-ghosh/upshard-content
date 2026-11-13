---
title: "Backend for Frontend and Gateway Patterns"
description: Why one API gateway serving every client type becomes a bottleneck, and how a tailored backend per client (BFF) fixes it.
roadmap: system-design
node: backend-for-frontend-and-gateway-patterns
group: architecture-patterns-at-scale
video:
  provider: youtube
  id: "b8wPnMtL_xU"
  start: 0
  end: null
creator:
  name: bulama io
  channel_url: https://www.youtube.com/@bulama
duration_seconds: 563
level: intermediate
prerequisites:
  - sidecar-ambassador-and-service-mesh
takeaways:
  - A single API gateway shared by every client type starts simple but accumulates client-specific branching logic for web, mobile, and third-party partners until it becomes a bottleneck every team has to coordinate through.
  - The backend-for-frontend (BFF) pattern gives each client type its own tailored backend, owned by the team that owns that client, instead of one general-purpose gateway serving all of them.
  - A BFF's job is shaping and aggregating data for one specific client — returning exactly the fields a mobile app's home screen needs in one trimmed response, instead of a generic payload the client has to filter down itself.
  - Gateway aggregation means combining several backend calls into one response at the edge, so a client makes one round trip instead of five — valuable on high-latency mobile networks especially.
  - Gateway offloading means moving concerns like authentication, TLS termination, and rate limiting out of individual services and into the shared edge layer, so every service doesn't reimplement them.
  - A BFF that starts serving every client and every partner, not just the one it was built for, has quietly turned back into the shared bottleneck it was meant to replace.
check_questions:
  - q: What specifically goes wrong with a single API gateway serving web, mobile, and third-party clients?
    a: The gateway accumulates branching logic and response shaping for each client's different needs (mobile wants a trimmed payload, web wants a richer one, partners need a stable versioned contract), and every team that wants to change their client's experience has to coordinate changes through that same shared component.
  - q: What does a backend-for-frontend actually do that a shared gateway doesn't?
    a: It's a backend built and owned for one specific client type, shaped entirely around that client's needs — aggregating and trimming data into exactly the response that client's screens require — rather than exposing one generic contract that every client type has to adapt to.
  - q: What's the difference between gateway aggregation and gateway offloading?
    a: Aggregation combines multiple backend calls into a single response at the edge so a client needs only one round trip; offloading moves shared infrastructure concerns like authentication, TLS termination, and rate limiting out of individual services and into the gateway layer so services don't each reimplement them.
related_formulas: []
---

## Why one gateway seems like the obvious answer

Once a system is split into several backend services, clients shouldn't have to know about all of them individually or call each one directly — that leaks internal service boundaries straight into every client and makes any internal refactor a client-facing breaking change. The standard fix is an **API gateway**: a single entry point that every client talks to, which routes requests to the right backend service, and centralizes concerns like authentication and rate limiting (see `reverse-proxies-and-api-gateways`). For a system with one client type, this works cleanly and is usually the right first move.

## Where the single gateway starts to strain

The trouble starts once a system has genuinely different client types with genuinely different needs: a web app that can afford a richer payload over a fast connection, a mobile app on a flaky network that wants the smallest possible response shaped exactly for one screen, and third-party partners who need a stable, versioned contract they're not going to adapt to every time the mobile team changes their home screen.

A single shared gateway serving all three ends up with branching logic baked into it: `if (client == mobile) trimFields(); if (client == partner) applyV2Contract();`. Every one of those branches is logic that the mobile team, the web team, and the partner-integrations team all now depend on inside a component none of them fully own. Changing the mobile app's home screen means touching a shared gateway that the web and partner teams also depend on — exactly the cross-team coordination bottleneck that splitting into services was supposed to avoid in the first place. Over time, the gateway becomes a second, informally-owned monolith sitting in front of the real one.

## The backend-for-frontend pattern

The **backend-for-frontend (BFF)** pattern's fix is to stop sharing one gateway across client types and instead give each client type its own backend, owned by the team that owns that client:

```
Mobile App  → Mobile BFF    ┐
Web App     → Web BFF       ├──→  shared backend services
Partner API → Partner BFF   ┘      (orders, catalog, users, ...)
```

Each BFF talks to the same underlying backend services, but shapes the result entirely around its one client. The mobile BFF can aggregate three backend calls — user profile, recent orders, recommended items — into the single trimmed JSON response the mobile home screen actually renders, cutting both the number of round trips and the bytes sent over a mobile network. The partner BFF can expose a deliberately conservative, versioned contract that changes on a slow, announced schedule, decoupled entirely from how often the mobile team ships. The mobile team can own and deploy the mobile BFF without asking the web or partner teams for a review, because nothing they change affects those other BFFs.

This isn't free: it's more services to run, and some logic (an auth check, a data-shaping rule) can end up duplicated across BFFs if teams aren't careful about which pieces genuinely belong in shared backend services versus which are legitimately client-specific. The pattern is worth it when client needs have actually diverged enough that a shared contract is causing friction — not as a default for every client a system happens to have.

## Gateway aggregation and offloading, in brief

Two capabilities commonly associated with gateways (and often implemented inside a BFF too) are worth knowing by name:

- **Aggregation** — combining several backend calls into one response at the gateway or BFF layer, so the client makes one round trip instead of orchestrating five itself. This matters most on high-latency connections, where five sequential mobile round trips can cost far more than the backend work those calls actually represent.
- **Offloading** — moving concerns that are genuinely the same across every backend service — TLS termination, authentication, rate limiting, request logging — out of each individual service and into the shared edge layer, so services don't each reimplement and maintain their own copy. This is the one thing that still make sense to centralize even once you've split gateways by client type, since authentication rules don't usually differ by client the way response shapes do.

## Keeping a BFF from becoming the next bottleneck

The failure mode that undoes the whole pattern is a BFF quietly growing to serve clients it wasn't built for — a "mobile BFF" that a partner integration starts calling too, or a second internal tool reusing the web BFF because it's convenient. The moment a BFF serves more than its one intended client, it has recreated the exact shared-bottleneck problem the pattern exists to avoid, just with an extra hop. Keeping each BFF scoped to exactly one client, with shared logic pushed down into the backend services it calls rather than duplicated sideways across BFFs, is what keeps the pattern paying off instead of multiplying the original problem.
