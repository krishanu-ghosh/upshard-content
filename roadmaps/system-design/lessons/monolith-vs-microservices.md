---
title: "Monolith vs. Microservices: The Real Trade-offs"
description: What splitting a monolith into microservices actually buys and costs you, and the concrete signals that tell you a system is ready to split.
roadmap: system-design
node: monolith-vs-microservices
group: architecture-patterns-at-scale
video:
  provider: youtube
  id: "lTAcCNbJ7KE"
  start: 0
  end: null
creator:
  name: ByteByteGo
  channel_url: https://www.youtube.com/@ByteByteGo
duration_seconds: 285
level: intermediate
prerequisites:
  - health-checks-and-chaos-engineering
takeaways:
  - A monolith is one codebase, one build, and one deployable unit; a function call to another module is free, synchronous, and type-checked by the compiler.
  - Splitting into microservices turns many of those in-process function calls into network calls, which can fail, time out, or arrive out of order — this is the single biggest hidden cost of the split.
  - The real payoff of microservices is independent deployability and fault isolation — one team ships without coordinating a release train, and one service crashing doesn't take the whole system down.
  - Microservices are not an unconditional upgrade — they trade a simple failure mode (the whole app is down) for a much larger number of smaller, harder-to-diagnose failure modes (which of twelve services is slow right now?).
  - Good signals to split are organizational (multiple teams blocked on the same deploy) or operational (one part of the system needs very different scaling or reliability), not simply that the codebase feels large.
  - A "distributed monolith" — services that are physically separate but still deploy in lockstep and share a database — gets microservices' network costs without getting any of the independence benefits.
check_questions:
  - q: What specifically turns into a network call when a monolith is split into microservices?
    a: A function call that used to execute in-process, synchronously, and with compiler-checked types now has to be serialized, sent over the network to another service, and handled for the case where it's slow, unreachable, or returns malformed data.
  - q: What is a "distributed monolith," and why is it worse than either a clean monolith or clean microservices?
    a: It's a system split into separate services that still have to be deployed together and usually share one database — it pays the full network, serialization, and operational cost of microservices while keeping the monolith's problem of every team being coupled to the same release.
  - q: Name one organizational signal and one operational signal that suggest a system is ready to split?
    a: Organizationally, multiple teams are repeatedly blocked waiting on each other's changes to the same deploy; operationally, one part of the system has a wildly different scaling or reliability profile than the rest, such as a reporting job that can tolerate being slow versus a checkout path that can't.
related_formulas: []
---

## What a monolith actually is

A monolith is a single codebase that builds into one deployable artifact and runs as one process (or one set of identical replicas behind a load balancer). When the checkout module needs the inventory module's `reserveStock()` function, it just calls it — in the same memory space, with the same transaction, checked by the same compiler. That call cannot time out, arrive twice, or fail halfway through in a way the language's own error handling doesn't already cover.

This is easy to dismiss as "legacy," but it's a genuinely strong default. A single deploy means a single thing to roll back. A single database means a single place transactions are consistent. A single process means a stack trace that spans the whole request, not just one hop of it.

## What splitting actually buys you

Microservices split that one codebase into several independently deployable services, each usually owning its own data. The real benefits are narrower than "it scales better":

- **Independent deployability.** The payments team can ship a fix on Tuesday afternoon without asking the catalog team to also be ready to deploy. In a monolith, one team's untested change can block everyone else's release.
- **Fault isolation.** If the recommendation service falls over, checkout can still work — it just stops showing recommendations. In a monolith, an unhandled exception or a memory leak in one module can take the entire process down.
- **Independent scaling.** A product search service that gets hammered on Black Friday can run on fifty replicas while the admin-reporting service runs on two, instead of scaling the whole monolith (and its heaviest, least-used parts) uniformly.
- **Technology and team boundaries.** A service can use the language, database, or runtime that fits its problem, and a team can own its service end to end — its own deploy pipeline, its own on-call, its own pace of change.

## What splitting actually costs you

None of this is free, and the costs are easy to underestimate before you've lived with them:

- **In-process calls become network calls.** Every call across a service boundary now has latency, can time out, can fail independently of the caller, and needs retry and idempotency handling that didn't exist before (see `idempotent-apis-and-exactly-once-semantics`). A request that touched three in-memory modules might now fan out across three network hops, each with its own failure mode.
- **Data consistency gets harder.** A monolith can wrap a multi-step update in one ACID transaction. Across services, you generally can't — you need patterns like sagas and compensating transactions, covered in full in `distributed-transactions-2pc-and-sagas`.
- **Observability has to be built, not inherited.** A single stack trace becomes a distributed trace across N services; you need centralized logging, correlation IDs, and dashboards just to answer "why was this request slow?"
- **Operational surface area multiplies.** Each service needs its own CI/CD pipeline, its own deployment config, its own health checks, its own dependency versions to patch. Ten services means ten times the infrastructure to keep healthy, not the same amount split ten ways.
- **Testing a feature that spans services is harder.** A monolith's integration test boots one process. A cross-service feature needs either a full staging environment with every dependent service running, or disciplined contract testing between them.

## The trap: a distributed monolith

The worst outcome isn't "monolith" or "microservices" — it's splitting a system into separate deployables that still have to be released together, still share one database, and still call each other synchronously in long chains. You've paid for the network calls, the serialization, and the extra operational surface, and gotten none of the independence that was supposed to justify it. If your "microservices" all break when any one of them is down, or you can't deploy service A without immediately deploying service B, you have a distributed monolith, and the honest fix is usually to either genuinely decouple the data and release process, or merge them back.

## Signals that a system is actually ready to split

Rather than splitting because microservices sound more modern, look for concrete pressure:

- **Team contention**, not codebase size: multiple teams are regularly blocked on each other's changes to the same deployable, and that's slowing releases down.
- **Divergent scaling or reliability needs**: one part of the system needs to scale or stay available in ways that would force you to over-provision the rest of the monolith to match it.
- **Divergent release cadence**: one part of the system needs to ship many times a day while another changes once a quarter, and bundling them into one release train is actively painful.
- **A genuinely separable data boundary**: the part you want to split out can own its own data without constant cross-boundary transactions back into the rest of the system.

If none of those are true yet, the complexity of a split is a cost you're paying for benefits you don't need yet — and a well-structured "modular monolith," with clear internal module boundaries but one deployment, often gets you most of the maintainability win without the network tax.
