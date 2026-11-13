---
title: "Stateless Services and Session Affinity"
description: Statelessness is what lets a load balancer treat instances as interchangeable; sticky sessions are the fallback when it's missing.
roadmap: system-design
node: stateless-services-and-session-affinity
group: scaling-data-and-services
video:
  provider: youtube
  id: "nFPzI_Qg3FU"
  start: 0
  end: null
creator:
  name: Hussein Nasser
  channel_url: https://www.youtube.com/@hnasr
duration_seconds: 884
level: intermediate
prerequisites:
  - load-balancing-algorithms
takeaways:
  - A stateless service instance keeps no memory of a client between requests, so any request from any client can be handled by any instance — that interchangeability is exactly what lets a load balancer route freely and what lets you add or remove instances without anyone noticing.
  - A stateful service instance holds something a specific client needs again later — an in-memory session, an uploaded file still being processed, a WebSocket connection — which ties that client to that one instance.
  - "Session state that can't live on the instance has to live somewhere every instance can reach: a shared cache like Redis, a database, or a signed token the client carries and presents on every request."
  - Sticky sessions (session affinity) are a load balancer feature that routes a given client to the same instance every time, which lets you keep state in memory without externalizing it — but at the cost of uneven load and a client losing their session if that one instance goes down.
  - "Externalizing state is usually the better fix: once the state lives outside any single instance, every instance becomes interchangeable again, and you get the scaling and failover benefits statelessness was providing in the first place."
  - Token-based authentication (like a JWT) is a common way to make a whole login system stateless — the server doesn't store session data at all, because every request carries enough signed information to verify itself.
check_questions:
  - q: Why can a load balancer freely send two requests from the same user to two different instances when the service is stateless, but not when it's stateful with in-memory sessions?
    a: A stateless instance doesn't keep anything specific to that user between requests, so every instance can handle every request identically; a stateful instance with in-memory sessions has that user's session data sitting only in one instance's memory, so a request routed to a different instance won't find it and the user effectively gets logged out or loses their state.
  - q: What's the actual cost of using sticky sessions instead of externalizing session state?
    a: Sticky sessions tie a client to one specific instance, which means load can become uneven if some clients are much more active than others, and if that one instance crashes or is taken down for a deploy, every client stuck to it loses their session — the fix trades operational flexibility for the simplicity of keeping state in memory.
  - q: If a service externalizes its session state into Redis, is it now "stateless"?
    a: The service instances themselves are stateless again — any instance can serve any request because none of them hold the session locally — but the system as a whole still has state; it's just been moved to a shared, centrally-reachable store instead of being pinned to one instance's memory.
deeper: []
related_formulas: []
---

## Why "interchangeable" is the property that matters

A load balancer's whole job gets dramatically simpler if every instance behind it is interchangeable — if it genuinely does not matter which instance handles the next request. That property is called **statelessness**: a stateless service instance doesn't keep anything in its own memory that depends on a specific client's history. Every request arrives with everything the instance needs to handle it, the instance does its work, and it forgets the client existed the moment it responds.

That's what makes horizontal scaling almost mechanical. Need more capacity? Start more instances — none of them need to "catch up" on anything, because there's nothing to catch up on. Need to deploy a new version? Kill an old instance and start a new one; no client was depending on that specific instance still being alive. A load balancer can use the simplest possible algorithm — round robin, least connections, whatever — because it isn't solving a harder problem underneath: it never has to guess which instance "remembers" a given client.

## Where state actually goes when it can't stay on the instance

The trouble is that real applications often do need to remember something about a specific client across more than one request: that they're logged in, what's in their cart, how far through a multi-step upload they've gotten. Statelessness doesn't make that need disappear — it just insists that memory can't live *only* inside one instance's process. It has to live somewhere every instance can reach equally:

- **A shared cache or database** — Redis is the common choice for session data specifically, because it's fast and every instance can read and write to it over the network. The instance handling a request looks up the session by an ID the client sends, rather than keeping it locally.
- **A token the client carries itself.** A signed JWT can embed the user's identity and permissions directly in the token, cryptographically verifiable by any instance that has the signing key — no shared store needed at all, because the "state" travels with the client on every request instead of sitting server-side.

```
Client request → Authorization: Bearer <JWT>
Instance verifies signature → trusts claims inside → no session lookup needed
```

Either approach gets you back to the same place: no single instance is the only place that knows about a given client, so any instance can serve any request, and the load balancer's job stays simple.

## Sticky sessions: the escape hatch, and its cost

Sometimes externalizing state isn't practical — a legacy application built assuming in-memory sessions, or a WebSocket connection that's inherently tied to the one instance holding it open. The load balancer-level answer to this is **session affinity**, usually called **sticky sessions**: the load balancer remembers which instance it sent a particular client to (often by setting a cookie, like `SERVERID=instance-7`) and routes every subsequent request from that client back to the same instance.

```
Client's first request  → LB picks instance-7 → sets cookie SERVERID=instance-7
Client's later requests → LB reads cookie    → routes back to instance-7 every time
```

This works, in the sense that in-memory state now reliably finds the same client again. But it quietly reintroduces most of what statelessness was avoiding. Load can become uneven — if a handful of clients are unusually active, the instances they're stuck to get disproportionately loaded while others sit idle, and the load balancer can't rebalance that away because it's committed to routing them consistently. Deploys and failures get more disruptive, too: taking instance-7 out of rotation, whether on purpose for a release or because it crashed, drops every session pinned to it, in a way that simply doesn't happen when any instance can serve any client.

## Why externalizing state usually wins

Sticky sessions are a reasonable short-term fix, especially for something you can't easily change (a third-party component, a connection that's genuinely instance-specific like a live WebSocket). But as a default architectural choice, externalizing state is almost always the better long-term answer, because it buys back everything statelessness was providing in the first place: instances become interchangeable again, the load balancer can use its simplest and most even routing strategy, scaling up or down stops being an event any client notices, and a crashed instance costs you nothing more than one in-flight request — not an entire session's worth of state. The work of externalizing state (standing up a shared cache, or switching to signed tokens) is a one-time cost; sticky sessions' unevenness and fragility are a cost you keep paying for as long as the system runs.
