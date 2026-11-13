---
title: "Choosing an Architectural Style"
description: A decision framework across monolith, microservices, SOA, serverless, and event-driven — which signals should actually drive the choice.
roadmap: software-architect
node: choosing-an-architectural-style
group: principles-and-patterns
video:
  provider: youtube
  id: "wUOGCN5P-to"
  start: 0
  end: null
creator:
  name: Code View And Talk
  channel_url: https://www.youtube.com/@CodeViewAndTalk
duration_seconds: 664
level: intermediate
prerequisites:
  - domain-driven-design
takeaways:
  - "Monolith, microservices, SOA, serverless, and event-driven are five points on a menu, not a maturity ladder — none of them is the \"advanced\" choice a system is supposed to graduate into."
  - "Conway's Law — a system ends up structured like the organization that built it — means team topology is as real a design input as any technical requirement, and ignoring it produces services that map to nobody's actual team boundaries."
  - "The deep mechanics of monolith-vs-microservices trade-offs (network calls replacing function calls, data consistency, operational surface area) are covered in system-design's own lesson; this lesson is about the wider menu and the signals that pick a style, not re-deriving that one axis."
  - "SOA and microservices both split a system into services, but SOA typically centralizes communication through a shared integration layer (see service-oriented-architecture-and-esb), while microservices favor decentralized, direct service-to-service communication with each service owning its own data."
  - "Serverless trades operational ownership for constraints — no servers to patch or scale, but cold starts, a vendor-specific execution model, and per-invocation pricing that can make steady, high-volume traffic more expensive than a provisioned alternative."
  - "Event-driven architecture isn't a competitor to the other four — it's a communication style that can run inside a monolith, a set of microservices, or a serverless system, built around services reacting to events rather than calling each other directly."
check_questions:
  - q: Why is "which architectural style is most advanced" the wrong question to ask?
    a: Each style trades a different set of costs for a different set of benefits — a monolith trades independent deployability for operational simplicity, serverless trades infrastructure ownership for cold starts and vendor lock-in — and a style that's right for a ten-person startup's MVP can be actively wrong for the same company at a thousand engineers, so "advanced" implies a ladder that doesn't actually exist.
  - q: How does Conway's Law act as a real constraint on architectural style, not just a saying?
    a: A team's existing structure and communication patterns tend to get reproduced in the software it builds regardless of what the architecture diagram says should happen, so choosing a highly decoupled microservices style for a tightly centralized team that makes every decision together tends to produce services that are "microservices" in name but still deploy and change together in lockstep.
  - q: What's the core structural difference between classic SOA and microservices, even though both split a system into services?
    a: SOA typically routes communication between services through a centralized integration layer like an enterprise service bus, which does protocol translation and routing in one place, while microservices favor each service talking directly to the services it depends on and owning its own data store, trading SOA's central coordination point for more services that each do less.
deeper:
  - label: "Monolithic vs Microservice Architecture: Which To Use and When?"
    url: "https://www.youtube.com/watch?v=NdeTGlZ__Do"
related_formulas: []
---

## A menu, not a ladder

It's tempting to read "monolith → microservices" as a progression every system eventually makes, the way a startup "graduates" from a garage to an office. That framing is wrong, and it's responsible for a lot of needlessly over-engineered systems. Monolith, microservices, SOA, serverless, and event-driven are five different points on a menu, each a genuine trade-off rather than a worse or better version of the others. The job of an architect choosing a style isn't picking the most sophisticated-sounding option; it's reading the actual signals a system and its team are giving off and matching the style to them.

## The menu, briefly

A **monolith** is one codebase, one deployable, one database — the strongest default for a new system because of how much it doesn't make you solve yet (see `system-design`'s `monolith-vs-microservices` for the full cost/benefit breakdown of this one axis; this lesson won't repeat it). **Microservices** split that system into independently deployable services, each usually owning its own data, communicating directly with each other over the network. **SOA** (service-oriented architecture) also splits a system into services, but the classic form routes that communication through a centralized integration layer — see `service-oriented-architecture-and-esb` for what that bus actually does and why SOA fell out of fashion relative to microservices. **Serverless** removes the server from the equation entirely: code runs as short-lived functions a cloud provider schedules, bills per invocation, and scales automatically — at the cost of cold starts, a provider-specific execution model, and a pricing curve that punishes steady, high-volume traffic. **Event-driven architecture** isn't on the same axis as the other four at all — it's a communication style, built around services publishing and reacting to events instead of calling each other directly, and it can run inside a monolith, inside a microservices fleet, or inside a serverless system equally well.

## The signals that actually matter

Four questions do most of the real work in picking a style:

- **Team topology.** This is **Conway's Law**: a system's structure tends to mirror the communication structure of the organization that built it, whether or not anyone designs it that way. A five-person team building a product end to end gets little real benefit from splitting it into twelve independently-deployed services — nobody's independence is actually being unblocked, because the same five people own all twelve. A thirty-person org split into five product teams, each blocked on the others for every release, is exactly the situation independent deployability exists to fix.
- **Change rate and blast radius.** A system where one part changes weekly and another changes once a year benefits from separating them, so a bug in the fast-moving part can't force a redeploy — or an outage — of the stable part. A system that changes uniformly across the board gets no such benefit from splitting.
- **Scaling profile.** If one part of a system needs ten times the compute of the rest under load — a checkout path during a sale, an image-processing pipeline — splitting that part out (as a service, or as a serverless function) lets it scale independently instead of forcing the whole system to over-provision to match its heaviest component.
- **Operational appetite.** Microservices and SOA both multiply the number of things that can independently fail and need monitoring, alerting, and on-call coverage. Serverless removes server management but introduces cold starts and a steep vendor-specific learning curve. An organization without the operational maturity (or headcount) to run either well will pay for that gap in outages, not in architecture-diagram elegance.

## A worked example

A ten-person startup building a new product should almost always start with a monolith: one deployable, one database, no network calls between modules that don't need to exist yet, and a small team that doesn't need independent release trains because everyone already talks to everyone. Three years later, with eighty engineers split into eight product teams, a shared checkout service blocking every team's release train on every deploy, and a recommendation engine that needs GPU-backed autoscaling the rest of the system doesn't, the signals have genuinely changed: team topology now calls for independent deployability, and the recommendation engine's scaling profile calls for pulling it out as its own service, quite possibly a serverless one given its bursty, event-triggered workload. Neither the ten-person version nor the eighty-person version was wrong — they were the right style for the signals present at the time, which is the actual skill this lesson is teaching: reading the signals again every time the system or the org changes shape, rather than picking a style once and defending it forever.
