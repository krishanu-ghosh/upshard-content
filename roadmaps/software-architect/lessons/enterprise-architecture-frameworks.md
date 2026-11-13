---
title: "Enterprise Architecture Frameworks"
description: What TOGAF's ADM cycle actually provides, and how enterprise architecture differs from solution and application architecture.
roadmap: software-architect
node: enterprise-architecture-frameworks
group: cloud-and-enterprise-patterns
video:
  provider: youtube
  id: "hhlxFtV_tZo"
  start: 0
  end: null
creator:
  name: GOTO Conferences
  channel_url: https://www.youtube.com/@GOTO-
duration_seconds: 3707
level: intermediate
prerequisites:
  - levels-of-architecture
takeaways:
  - "An enterprise architecture framework exists to answer questions no single application or solution architect is positioned to answer — where does this capability live across the whole organization, which of our twelve CRMs is the one new systems should integrate with, what's the five-year target state for data ownership."
  - "TOGAF's Architecture Development Method (ADM) is an iterative cycle, not a one-time document: it moves from an architecture vision through business, data, application, and technology architectures, to opportunities and migration planning, to implementation governance, and back around as the organization changes."
  - "A framework like TOGAF deliberately does not prescribe which cloud pattern, which database, or which programming language to use — it prescribes a process for deciding and documenting those things consistently across dozens of teams and systems that would otherwise each invent their own."
  - "Enterprise architecture's real, durable product is usually not a static diagram but an architecture practice — a repeatable way for a large organization to make and record compatible technology decisions over years, as people and projects turn over."
  - "The honest critique of heavyweight EA frameworks is real: applied as a rigid, binder-filling compliance exercise, TOGAF becomes governance theater; applied as a tailored, lightweight checklist for the decisions that genuinely need enterprise-wide coordination, the same framework earns its keep."
check_questions:
  - q: What question does enterprise architecture answer that a single application's architect typically cannot?
    a: Whether a given capability, system, or data model should be shared, duplicated, or consolidated across the whole organization — a decision that requires visibility into every other system and team, not just the one application an individual architect is responsible for.
  - q: Why is TOGAF's Architecture Development Method described as an iterative cycle rather than a one-time deliverable?
    a: Because the organization, its systems, and its priorities keep changing after the first pass through the ADM's phases, so the method is designed to be run again — revisiting the vision, the target state, and the migration plan — rather than producing one architecture document that's assumed to stay correct indefinitely.
  - q: In what sense does TOGAF deliberately avoid prescribing specific technical choices?
    a: It defines a process for how an organization should go about deciding things like which cloud pattern or data platform to standardize on and how to document and govern that decision consistently, but it leaves the actual technical choice itself up to the organization using it, since the right answer differs by context.
deeper: []
related_formulas: []
---

## A different scope than the rest of this roadmap

Every other lesson in this roadmap operates at the scale of one system: how do you structure its codebase, choose its data model, decide whether it should be a monolith or several services. **Enterprise architecture** operates one level above that — across every system an organization runs, over a multi-year horizon, answering questions no single system's architect is positioned to answer on their own: which of the organization's several CRMs should new integrations target, where does "customer" get defined once so twelve systems stop disagreeing about it, what's the plan for retiring the mainframe over the next five years. An **enterprise architecture framework** is a structured method for making and documenting that category of decision consistently, instead of leaving each team to improvise its own answer and quietly diverge from every other team's.

## TOGAF and the Architecture Development Method

**TOGAF** (The Open Group Architecture Framework) is the most widely adopted framework in this space, and its core is the **Architecture Development Method (ADM)** — an iterative cycle of phases rather than a single document you write once and file away. In rough order, it moves through: a **preliminary phase** that sets up the organization's architecture capability and principles; an **architecture vision** phase that defines scope and gets stakeholder buy-in for a specific initiative; **business, data, application, and technology architecture** phases that each work out the target state for that layer; an **opportunities and solutions** phase that identifies the concrete projects needed to get there; **migration planning** that sequences those projects realistically against budget and dependencies; and **implementation governance** that checks delivered work against the architecture as it's actually built. Crucially, the cycle loops back to the start — new business priorities or technology shifts trigger another pass through the same phases, which is why the ADM is better understood as an operating rhythm for an architecture practice than as a waterfall process with a finish line.

## What the framework prescribes, and what it deliberately doesn't

A common misreading of TOGAF is to expect it to tell you which database, which cloud provider, or which integration pattern to use — it doesn't, and that's by design. TOGAF prescribes the **process**: how to capture the current state, define a target state, identify the gap, and govern the transition, consistently, across however many teams and systems an organization has. The actual technical decisions inside that process — pick Postgres or a document store, standardize on REST or GraphQL, go all-in on one cloud or stay multi-cloud — are left to the organization, informed by the same kind of trade-off reasoning this roadmap's other lessons teach at the system level. This is also where TOGAF connects back to the levels this roadmap opened with: enterprise architecture sets the target state and the constraints; solution and application architecture (the levels most of the rest of this roadmap operates at) work out how a specific system meets them.

## The real product is a practice, not a diagram

The artifacts an EA initiative produces — current-state diagrams, target-state diagrams, a roadmap of migration projects — are easy to mistake for the point. The more durable value is the **practice** itself: a repeatable way for decisions to get made and recorded so that five years later, after half the original team has moved on, someone can still answer "why does this system own customer identity, and what's it supposed to hand off to that one once the migration finishes?" without that knowledge having lived only in one architect's head. A framework that produces a beautiful one-time diagram but no ongoing practice for keeping decisions coherent as the organization changes hasn't actually delivered the thing enterprise architecture exists to provide.

## Where the common criticism is fair, and where it isn't

TOGAF and its peers get a well-earned reputation for bureaucracy, and the criticism is fair in a specific, common failure mode: an organization adopts the framework as a compliance exercise, produces binders of documentation that nobody downstream reads or updates, and gates every decision through a review board slow enough that teams route around it entirely. That's governance theater, not architecture. The same framework, tailored down to the decisions that genuinely benefit from enterprise-wide coordination — shared identity, a small number of blessed data platforms, a consistent integration pattern — and left out of decisions that don't need it, is the version that actually earns its keep. The lesson for an architect evaluating whether to introduce (or push back on) a framework like this isn't "TOGAF good" or "TOGAF bad" — it's to ask, for this specific organization, which decisions are costly enough when made inconsistently that a shared process is worth the overhead, and to scope the framework to exactly those.
