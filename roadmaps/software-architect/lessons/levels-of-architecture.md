---
title: "Levels of Architecture — Enterprise, Solution, and Application"
description: The three scopes architects work at, how the questions and stakeholders change at each one, and why mixing them up causes real damage.
roadmap: software-architect
node: levels-of-architecture
group: the-architect-role
video:
  provider: youtube
  id: "LttxTgZkKIk"
  start: 0
  end: null
creator:
  name: Go Cloud Architects
  channel_url: https://www.youtube.com/@GoCloudArchitects
duration_seconds: 638
level: beginner
prerequisites:
  - what-is-software-architecture
takeaways:
  - "Application architecture is the narrowest scope — the internal structure of one system or service, owned by one team, with decisions like layering, module boundaries, and the patterns in `architectural-patterns-and-dependency-injection`."
  - "Solution architecture sits above a single application — it designs how several systems, teams, and sometimes third-party services combine to deliver one business capability end to end, such as checkout or onboarding."
  - "Enterprise architecture is organization-wide — standards, shared platforms, technology portfolios, and data governance that apply across every solution, usually owned by a small team with a multi-year horizon rather than a single project's."
  - "The three levels differ in questions asked, not just scope — application architecture asks how is this one system built, solution architecture asks how do these systems work together to solve this business problem, and enterprise architecture asks what should every team be standardizing on."
  - "Mixing levels causes real damage both directions — an application architect imposing enterprise-wide standards without solution-level context creates rules nobody's actual system fits, while an enterprise architect skipping straight to implementation detail produces policy that ignores what any one team actually needs."
check_questions:
  - q: What distinguishes a solution-architecture decision from an application-architecture one?
    a: Application architecture is scoped to one system's internal structure, owned by one team; solution architecture is scoped to how multiple systems, teams, and often external services combine to deliver one end-to-end business capability.
  - q: Why does enterprise architecture typically operate on a longer time horizon than solution or application architecture?
    a: It sets standards and shared platforms meant to serve every current and future solution across the organization, not one project's delivery timeline, so its decisions are judged by multi-year consistency and reuse rather than a single release.
  - q: What's one concrete failure mode of an enterprise architect skipping straight to implementation-level decisions?
    a: They produce a mandated standard — a specific framework, data format, or tool — that ignores the real constraints of the individual teams who have to adopt it, creating policy that looks correct at the enterprise level but doesn't fit how any one system actually works.
deeper:
  - label: "Solutions vs Enterprise Architecture: Understanding the Key Difference"
    url: "https://www.youtube.com/watch?v=Ksd0bT2Vx-Q"
related_formulas: []
---

## Three scopes, three different jobs

"Architect" is one word doing the work of at least three different jobs, and most of the confusion around the title — what an architect is supposed to do, who they report to, what decisions are theirs to make — comes from not naming which of the three scopes is actually in play.

**Application architecture** is the narrowest: the internal structure of one system, owned by one team. Should this service be layered or hexagonal? Where does validation logic live? How is dependency injection wired up? These are exactly the questions `architectural-patterns-and-dependency-injection` (later in this roadmap) covers in depth, and they're answerable by looking at one codebase in isolation.

**Solution architecture** is one level up: how several applications, teams, and often third-party services combine to deliver one business capability end to end. "Checkout" isn't one application — it's a payments service, an inventory service, a notification system, a fraud-check vendor, and a frontend, all owned by different teams, that together have to behave as one coherent flow. A solution architect's job is designing and defending the contracts between those pieces: who calls whom, synchronously or async, what happens when one piece is down, who owns which piece of shared data.

**Enterprise architecture** is organization-wide: the standards, shared platforms, and technology portfolio every team is expected to build on, regardless of which solution they're working on. Which cloud provider is approved. Which identity provider every service authenticates against. Whether teams may stand up a new database technology or must use one of three sanctioned ones. An enterprise architect isn't designing any one system — they're designing the constraints every system has to work inside, usually justified by economies of scale (one identity system instead of fifteen), risk management (one audited data-handling policy instead of each team inventing its own), or genuine cross-team reuse.

## Same word, different questions

The cleanest way to tell the levels apart isn't org-chart position, it's the question each one is actually trying to answer:

- Application architecture asks: **how is this one system built**, such that it's maintainable, testable, and correct?
- Solution architecture asks: **how do these systems work together** to deliver one business outcome, given that each is owned by a different team with its own release cadence and failure modes?
- Enterprise architecture asks: **what should every team be standardizing on**, so the organization isn't paying for N incompatible solutions to the same problem?

None of the three is "more architectural" than the others — they're orthogonal scopes, and a single person can move between them on different projects, or even within the same week. What doesn't work is answering one level's question with another level's tools.

## What goes wrong when the levels get mixed up

Mixing levels fails in both directions, and both failure modes are common enough to be worth naming explicitly.

**Enterprise-down failure:** an enterprise architect mandates a specific technology or pattern without solution-level context — "every service must use this particular message queue" — and the mandate turns out to be a poor fit for a solution with genuinely different latency or ordering requirements. The standard was reasoned about correctly at the enterprise level (fewer technologies to operate and secure) but applied as if it were an application-level decision, ignoring the actual constraints of the system it's imposed on.

**Application-up failure:** a team solves a problem at application scope that was actually a solution- or enterprise-scope problem in disguise. A service picks its own authentication scheme because it's the fastest path to shipping, without noticing that four other services in the same business flow each made a different, incompatible choice — now the "solution" is five applications that each work individually and don't actually integrate, and someone has to retrofit a shared identity layer after the fact, at a much higher cost than designing it in from the solution level up front.

## Why this matters before the rest of the roadmap

Most of the lessons that follow — architecture decision records, SOLID, domain-driven design, choosing an architectural style — are tools an architect reaches for, and which tool is right depends partly on which of these three levels the decision actually lives at. A trade-off worth recording in an ADR at the application level (how one service wires its dependencies) is a very different conversation from one at the enterprise level (which cloud region every team's data lives in, for compliance reasons). Naming the level first is usually the fastest way to figure out who actually needs to be in the room for a given decision.
