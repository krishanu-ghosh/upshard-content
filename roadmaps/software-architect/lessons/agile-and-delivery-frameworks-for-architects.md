---
title: "Architecture Under Agile Delivery — What Actually Changes"
description: Where big design up front fails under iterative delivery, where some up-front design still has to survive, and how decisions actually get fed into sprints.
roadmap: software-architect
node: agile-and-delivery-frameworks-for-architects
group: the-architect-role
video:
  provider: youtube
  id: "8GONv6jJsG0"
  start: 0
  end: null
creator:
  name: Modern Software Engineering
  channel_url: https://www.youtube.com/@ModernSoftwareEngineeringYT
duration_seconds: 1117
level: intermediate
prerequisites:
  - managing-technical-debt-and-evolutionary-architecture
takeaways:
  - "\"Big design up front\" fails under iterative delivery for a specific reason, not just because agile frameworks dislike planning — early decisions are made with the least information you'll ever have, and locking them in before any real feedback arrives front-loads the cost of being wrong."
  - "The honest alternative isn't \"no design,\" it's enough design up front to make the first few iterations safe, followed by architecture that continues to be shaped as real feedback arrives — evolutionary architecture (the previous lesson) is what makes that safe rather than chaotic."
  - "Some decisions genuinely cannot be made incrementally without serious cost — a core data model, a choice of synchronous versus event-driven integration, which organization owns which bounded context — and pretending otherwise just defers an expensive decision to a moment when more code already depends on the wrong answer."
  - "Scrum and Kanban don't define an architect role at all, which is a gap teams have to fill deliberately rather than a signal that architecture isn't needed — without a deliberate answer, architectural decisions either get made by default inside a sprint by whoever is under the most delivery pressure, or get silently skipped."
  - "A workable pattern many teams converge on independently is a short architectural runway maintained slightly ahead of current sprint work — not a separate up-front phase, and not zero design, but a standing habit of keeping the next one or two increments' worth of hard decisions resolved before the team needs them."
check_questions:
  - q: What's the actual argument against big design up front, beyond "agile frameworks are against planning"?
    a: Early in a project you have the least real information you'll ever have about how the system will actually be used, so committing to a large, detailed design before any real usage or feedback exists locks in decisions made under maximum uncertainty, and the cost of being wrong compounds the longer the design goes unchallenged.
  - q: Why can't every architectural decision be made incrementally, one sprint at a time?
    a: Some decisions — a core data model, a synchronous-versus-event-driven integration style, which team owns which part of the domain — are expensive enough to change that building around the wrong one for several sprints before noticing costs far more than taking the time to get it right before code depends on it.
  - q: Why is the lack of a defined architect role in Scrum a problem worth solving deliberately?
    a: Without an explicit answer for who owns cross-cutting architectural decisions, those decisions don't disappear — they get made implicitly, inside a sprint, by whoever is under the most delivery pressure at the time, which is a worse process than deciding on purpose who makes the call and when.
deeper:
  - label: "Lesson 30 - Agile and Architecture"
    url: "https://www.youtube.com/watch?v=U36JoWr2rDk"
related_formulas: []
---

## Why big design up front actually fails

"Big design up front" (BDUF) earned its bad reputation honestly: a project spends weeks or months producing a detailed design before writing meaningful code, based entirely on assumptions about how the system will be used — assumptions that are, at that point in the project, at their least informed. By the time real usage, real data, and real user feedback arrive, several of those assumptions are usually wrong, and the team is now stuck either reworking an expensive, heavily-documented design or shipping something that doesn't actually fit what was learned. The failure isn't that planning is bad — it's that committing deeply to decisions before the information needed to make them well actually exists front-loads the cost of being wrong to the moment you can least afford it.

The honest fix isn't "no design." Dave Farley, whose whole career argument is for disciplined engineering practice rather than against planning, frames it as doing **enough** design up front to move forward safely, while deliberately leaving room to learn and adapt as real feedback comes in. That's a different claim from "architecture emerges entirely from code" — it's closer to: decide what you can't safely defer, and defer everything else until you have better information to decide it with.

## What genuinely can't be incremental

Not every decision benefits from being made just-in-time, and pretending otherwise is its own failure mode. Some decisions are expensive enough to reverse that discovering the wrong answer three sprints in costs far more than taking the time to get it right before any code depends on it:

- **Core data model.** If the central entities and their relationships are wrong, every feature built on top inherits the wrong shape, and untangling it later means migrating data, not just refactoring code.
- **Integration style.** Whether two parts of the system talk synchronously or through events changes the failure modes, the testing strategy, and the operational model for everything built on top — switching later is closer to a rewrite than a refactor.
- **Bounded-context ownership.** Which team owns which part of the domain shapes org structure, on-call responsibilities, and API contracts; getting it wrong means renegotiating ownership after teams have already built habits and expectations around the original split.

The discipline evolutionary architecture and fitness functions (the previous lesson) provide is exactly what makes the *rest* of the design safe to leave incremental: once the handful of genuinely hard-to-reverse decisions are made deliberately, everything else can evolve under automated guardrails instead of needing to be pre-decided out of caution.

## The role Scrum and Kanban don't define

Neither Scrum nor Kanban specifies an architect role, a fact that surprises teams adopting one of them for the first time. This isn't an oversight that needs correcting by adding a ceremony — it's a deliberate gap, because these frameworks describe how work flows through a team, not who owns which category of decision. Left unfilled, the gap doesn't mean architecture stops mattering; it means architectural decisions get made anyway, just implicitly, by whoever is under the most delivery pressure when a cross-cutting question comes up mid-sprint — usually without the trade-off analysis or documentation `architecture-decision-records-and-tradeoffs` argues for, because nobody was designated as responsible for doing it on purpose.

## A pattern that actually works inside sprints

Teams that handle this well tend to converge independently on the same shape, often called an **architectural runway**: a standing habit of keeping the next one or two increments' worth of hard architectural decisions resolved slightly ahead of the sprint that will need them, rather than discovering mid-sprint that a story can't start until a cross-cutting decision gets made. This isn't a separate up-front design phase bolted onto the front of the project — it's a continuous, lightweight practice, often just an architect or two spending a fraction of their time one or two sprints ahead of the delivery team, surfacing and resolving the decisions in time for the team that needs them, writing each one down as an ADR as it's made.

The practical shift this requires from a traditional, phase-gated mindset: architecture is never "done" and handed off before implementation starts, but it's also never purely reactive, invented story by story with no look-ahead at all. It's a continuous, running activity that stays just far enough ahead of delivery to keep the team from stalling on undecided cross-cutting questions, while leaving the decisions that genuinely can wait for real feedback to actually wait for it.
