---
title: "Domain-Driven Design"
description: Bounded contexts, ubiquitous language, and aggregates — the strategic and tactical halves of DDD, and why they decide where service boundaries go.
roadmap: software-architect
node: domain-driven-design
group: principles-and-patterns
video:
  provider: youtube
  id: "4j71Toy5ZLw"
  start: 0
  end: null
creator:
  name: CodeOpinion
  channel_url: https://www.youtube.com/@CodeOpinion
duration_seconds: 420
level: intermediate
prerequisites:
  - solid-principles
takeaways:
  - "Domain-Driven Design splits into a strategic half — how a large domain is divided into bounded contexts — and a tactical half — the patterns (aggregates, entities, value objects) used to model inside one context."
  - "A bounded context is a boundary within which one model and one shared vocabulary apply consistently; the word \"order\" can mean something different in the shipping context than it does in the billing context, and DDD says that's fine as long as each context is explicit about its own meaning."
  - "Ubiquitous language is the discipline of using the domain experts' own vocabulary in code, variable names, and conversation, rather than translating it into generic technical terms that lose meaning at the boundary between business and engineering."
  - "An aggregate is a cluster of entities and value objects treated as one unit for the purpose of data changes — it has one root entity that's the only object outside code is allowed to hold a reference to, which is what keeps the aggregate's internal invariants from being violated by code that reaches in and edits a child object directly."
  - "Bounded contexts are the natural unit to split a system into services, because they're already drawn around where a vocabulary and a model stay consistent — splitting a monolith along any other line tends to cut through a model that was never meant to be divided."
check_questions:
  - q: Why can "order" legitimately mean different things in a shipping context and a billing context within the same system?
    a: Each bounded context has its own model built for its own purpose — shipping cares about an order's weight, address, and delivery status, while billing cares about its line items and payment state — and DDD treats this as healthy specialization rather than an inconsistency to resolve, as long as each context is explicit about what it means by the term.
  - q: What problem does ubiquitous language solve that a purely technical vocabulary doesn't?
    a: When code uses terms like "record" or "entity" instead of the domain experts' own words like "claim" or "policy," every conversation between engineers and domain experts requires a mental translation step, and that translation is exactly where requirements get lost or misunderstood; using the experts' real vocabulary in the code removes the translation step entirely.
  - q: Why can't code outside an aggregate hold a direct reference to one of its child entities?
    a: The aggregate root is the only object the aggregate guarantees is consistent after every change — a line item inside an Order aggregate might need the order's total recalculated whenever it changes, and that only happens reliably if every change is routed through the Order root's own methods, never by editing the line item directly from outside.
deeper:
  - label: "Domain-Driven Design in 150 Seconds"
    url: "https://www.youtube.com/watch?v=8Z5IAkWcnIw"
related_formulas: []
---

## Two halves: strategic and tactical

Domain-Driven Design, introduced by Eric Evans in 2003, is usually taught backward — tactical patterns first, strategic design second — which is exactly the wrong order for an architect. The **tactical** half is the set of code-level patterns (entities, value objects, aggregates, repositories) used to model one part of a business domain. The **strategic** half is about something an architect decides before any of that code gets written: how a large, messy domain gets divided into smaller, coherent pieces in the first place. Strategic design is the part that actually shapes a system's architecture; tactical patterns are how you build well inside whichever piece you're standing in.

## Bounded contexts: one model, one vocabulary, one boundary

A **bounded context** is the boundary within which a single model and a single shared vocabulary stay consistent. Take the word "order" in an e-commerce system. In the shipping context, an order is a set of items with a weight, a destination address, and a delivery status. In the billing context, the same word means a set of line items, a payment method, and a balance due. These aren't competing, inconsistent definitions that need to be unified into one canonical `Order` class — they're two different models, each fit for its own purpose, and DDD's actual claim is that forcing them into a single shared model is what causes real pain: one bloated `Order` class with fields that only half its callers need, changed constantly because every team touches the same object for unrelated reasons. A bounded context makes that boundary explicit, so "order" in shipping's code and "order" in billing's code are allowed to diverge on purpose.

## Ubiquitous language: the vocabulary is the interface

**Ubiquitous language** is the practice of using a domain expert's actual words — not a generic technical translation of them — consistently in conversation, in documentation, and in the code itself. If an insurance domain expert talks about a "claim" moving through "adjudication," the code should have a `Claim` class with an `adjudicate()` method, not a generic `Record` with a `processStatus` field that an engineer has to mentally map back to "claim" and "adjudication" every time they read it. The payoff isn't aesthetic. Every time code uses a different word than the business does, there's a translation step, and translation steps are where requirements silently drift — an engineer implements what they think "adjudication" means rather than what it actually means, because the code never forced them to use the real word and think in the real concept.

## Aggregates: one root, one set of invariants

Inside a bounded context, an **aggregate** is a cluster of related entities and value objects that must stay consistent as a unit. An `Order` aggregate might contain the order itself (the root) plus a collection of `LineItem` entities. The rule that makes this work is that outside code is only ever allowed to hold a reference to the aggregate root — never to a `LineItem` directly. If code could reach past the root and edit a line item's quantity directly, nothing would recalculate the order's total or re-check whether the order still qualifies for free shipping; the aggregate's internal invariants would silently drift out of sync. Routing every change through `Order.addLineItem(item)` or `Order.removeLineItem(id)` means the root can enforce its own rules on every single mutation, which is the entire reason an aggregate exists as a concept rather than just being "a few related classes."

## Why bounded contexts are where service boundaries belong

The strategic half of DDD matters most when a system is being split into services, because a bounded context is already drawn exactly where a system's natural seams are — the places where vocabulary, model, and ownership can genuinely diverge without constant cross-talk. Splitting a monolith along an arbitrary technical line (all the database access code in one service, all the HTTP handlers in another) tends to cut straight through a model that was never meant to be divided, forcing the resulting services into constant, chatty communication just to reconstruct what used to be one coherent object. Splitting along bounded-context lines instead means each resulting service owns a model that was already self-contained, which is why DDD's strategic half is often cited as the right starting point for deciding where `choosing-an-architectural-style`'s service boundaries should actually fall, rather than guessing from infrastructure concerns alone.
