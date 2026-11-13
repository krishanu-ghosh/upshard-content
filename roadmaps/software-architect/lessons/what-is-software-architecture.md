---
title: "What Is Software Architecture, and What Does an Architect Actually Do"
description: What separates an architectural decision from an implementation one, and what the architect role does differently from a senior engineer.
roadmap: software-architect
node: what-is-software-architecture
group: the-architect-role
video:
  provider: youtube
  id: "3fY86tUF0GY"
  start: 0
  end: null
creator:
  name: Software Architecture Monday
  channel_url: https://www.youtube.com/@markrichards5014
duration_seconds: 551
level: beginner
prerequisites: []
takeaways:
  - "An architectural decision is one that is expensive to reverse and affects the whole system — not a stylistic choice inside one class or module, but something every later decision has to be made around."
  - "The architect role exists to own decisions that no single team can safely make on its own, because getting one wrong is costly to undo once the rest of the system has grown around it."
  - "Day to day, the job leans toward trade-off analysis, documentation, and cross-team conversation more than it leans toward writing code — deciding and communicating is the output, not a working feature."
  - "\"Architect\" is a set of responsibilities an experienced engineer takes on, not a rank above engineering — the skill is the same engineering judgment, aimed at a wider and more irreversible scope of decisions."
  - "A good architect keeps options open as long as it is responsible to, deferring the genuinely irreversible calls until there is enough real information to make them well."
check_questions:
  - q: What makes a decision "architectural" rather than just an implementation detail?
    a: It is hard and expensive to reverse once the system has grown around it, and it affects more than one team or module — picking a datastore or a service boundary, not which library formats a date.
  - q: Why can't architectural decisions just be made locally by whichever team gets there first?
    a: Because the decision constrains every other team's future options, not only the team that made it — left local, it risks solving one team's immediate problem while quietly creating a system-wide one nobody chose on purpose.
  - q: How does an architect's day-to-day work differ from a senior engineer who isn't doing architecture?
    a: More time goes into comparing trade-offs, writing decisions down, and aligning several teams around a shared direction, and less time goes into shipping production code personally.
deeper: []
related_formulas: []
---

## The test for "architectural"

Every codebase is full of decisions, and most of them don't need an architect. Whether a function takes two arguments or a struct, whether a loop is a `for` or a `map`, which internal helper a module exposes — these are cheap to change. Get one wrong and a single pull request fixes it tomorrow.

A decision becomes architectural when it fails that test: when reversing it later means rewriting a large fraction of the system, migrating data, renegotiating a dozen teams' interfaces, or eating months of downtime. Choosing a relational database over a document store, splitting a monolith into services, picking a synchronous or event-driven integration style, deciding which team owns which part of the domain — these decisions don't just solve today's problem, they set the shape every future decision has to fit inside. That is the one-sentence definition worth keeping: **architecture is the set of decisions that are hard to change later and that everything else gets built on top of.** Everything else — patterns, principles, specific technology choices — exists in service of making that one kind of decision well.

## Why the role exists at all

If every engineer is capable of good judgment, why carve out a distinct role for this? Because the hard part of an architectural decision usually isn't technical skill, it's scope. A team optimizing its own service can make a locally excellent choice that is a system-wide liability — a message format that's perfect for one consumer and unreadable by the other five, a caching strategy that solves one team's latency problem by quietly breaking another team's consistency guarantees. Someone has to hold the whole picture, weigh trade-offs that cross team boundaries, and be accountable when two good local decisions conflict globally. That's the gap the architect role fills: not superior technical ability, but a different scope of responsibility and the authority (and the accountability) that has to come with it.

This is also why "architect" is badly understood as a promotion. It's a specialization, the way "SRE" or "staff engineer focused on data" are specializations — a senior engineer choosing to spend their judgment on system-wide, hard-to-reverse decisions instead of on deep ownership of one service or feature area. The skill underneath is the same engineering judgment a strong senior engineer already has; what changes is the altitude it's applied at.

## What the job actually looks like day to day

The popular image of an architect — drawing boxes and arrows all day, dictating technology choices from a tower — is mostly wrong, and it's a sign of a bad architect where it's true. In practice the job looks like:

- **Trade-off analysis.** Most architectural questions don't have a single correct answer, only a set of costs and benefits that land differently depending on the organization's actual constraints — team size, latency requirements, compliance obligations, how much the business is willing to pay in operational complexity for independence between teams. The architect's job is making that trade-off explicit rather than letting it default to whoever argued loudest in a meeting.
- **Writing it down.** A decision that lives only in one meeting's memory gets re-litigated every few months by whoever wasn't in the room. `architecture-decision-records-and-tradeoffs`, the next lesson in this group, covers the concrete artifact — the ADR — that architects use to make a decision and its reasoning durable.
- **Cross-team facilitation.** Many architectural decisions sit at the seam between two or more teams, and the architect is often the only person with standing to insist those teams actually negotiate the interface between them instead of each building to their own assumptions.
- **Owning non-functional requirements.** Features ship with functional requirements built in — what the product has to do. Architecture is largely about the requirements nobody puts in a user story: how available the system needs to be, what it costs to run, how it fails safely, how it scales. Someone has to be the one asking those questions before they become incidents.
- **Staying close enough to the code to be credible.** An architect who hasn't touched the codebase in two years starts making decisions based on how the system used to work, not how it works now — the role requires enough ongoing hands-on involvement to keep that picture current, even as the proportion of time spent writing code personally goes down.

## A role, not a rank

Gregor Hohpe, who spent years architecting at both Google and AWS, frames the job's real measure as whether it makes everyone else on the team smarter and more effective — not whether the architect personally made the cleverest call in the room. That's a useful corrective to the "smartest person in the room" caricature: the architect's output isn't a diagram, it's a system that the rest of the organization can build on confidently, and a set of decisions everyone understands well enough to extend correctly without having to ask. The rest of this roadmap is about the specific tools — levels of scope, decision records, principles, patterns, and the paradigms an architect has to reason across — that make that possible.
