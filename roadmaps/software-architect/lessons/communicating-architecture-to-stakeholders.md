---
title: "Communicating Architecture to Stakeholders Who Don't Read Diagrams"
description: Why most architecture diagrams fail their audience, and how the C4 model's four levels fix that by matching detail to who's actually looking.
roadmap: software-architect
node: communicating-architecture-to-stakeholders
group: the-architect-role
video:
  provider: youtube
  id: "lrTbmFhjv-4"
  start: 0
  end: null
creator:
  name: NLJUG
  channel_url: https://www.youtube.com/@NLJUG1
duration_seconds: 3042
level: intermediate
prerequisites:
  - architecture-decision-records-and-tradeoffs
takeaways:
  - "Most architecture diagrams fail not because they're wrong, but because they mix abstraction levels on one page — boxes for whole systems next to boxes for individual classes, which forces every reader to guess which zoom level they're looking at."
  - "The C4 model fixes this by defining four fixed levels — Context, Container, Component, and Code — each a consistent zoom level, so a diagram never has to serve two audiences' detail needs at once."
  - "Context diagrams are for non-technical stakeholders: your system as one box, the people and other systems around it, no internals at all — exactly the altitude a product sponsor or exec actually needs."
  - "Container and Component diagrams are for engineers: Container shows the deployable pieces (services, databases, the frontend) and how they talk; Component shows what's inside one container, without dropping to the level of a UML class diagram."
  - "The deepest level, Code, is intentionally the one C4 spends the least effort on — Simon Brown's own view is that a generated class diagram or the code itself already documents this level better than a hand-maintained one ever will."
check_questions:
  - q: Why does a single architecture diagram that mixes a whole subsystem and an individual class in the same box confuse its audience?
    a: It forces every reader to silently figure out which zoom level each box represents, so a non-technical stakeholder and an engineer reading the same diagram come away with different, often wrong, ideas of what it's actually showing.
  - q: What should a Context-level C4 diagram show, and who is it for?
    a: Your system as a single box, surrounded by the people and other systems it interacts with, with no internal structure shown at all — it's aimed at non-technical stakeholders who need to know what the system does and touches, not how it's built.
  - q: Why does C4 deliberately put the least effort into its deepest level, Code?
    a: Because a class diagram generated from, or simply read directly out of, the actual code is more accurate and far cheaper to keep current than a hand-drawn one that will drift out of date the moment the code changes.
deeper: []
related_formulas: []
---

## Diagrams fail for a specific, fixable reason

Ask five engineers to draw "the architecture" of the same system and you'll typically get five incompatible diagrams — not because they disagree about the system, but because nobody agreed on what altitude the diagram is drawn at. One engineer draws every microservice as a box; another draws every class; a third mixes both on the same page, because the one class that matters most happens to feel architecturally significant to them in the moment. The reader — who is very often not an engineer at all — has no way to tell which boxes represent "a whole deployable system" and which represent "one internal module," and ends up either overwhelmed by detail that wasn't meant for them or under-informed because the one box that actually mattered to their question got lost among forty others at the wrong zoom level.

This is the actual problem Simon Brown's **C4 model** solves, and it's a communication model before it's a notation. The insight is simple: fix the zoom levels in advance, one diagram per level, and never let a single diagram answer two different altitude questions at once.

## The four levels

**Context** is the widest zoom: your system as a single, undifferentiated box, surrounded by the people (user types, roles) and other systems it talks to. No internal structure appears at all — not even "frontend" and "backend." This is the diagram for a non-technical stakeholder, an executive sponsor, or a new engineer's first five minutes on the project: what does this thing do, and what does it touch? It answers exactly the altitude of question those audiences actually have, and nothing more.

**Container** zooms in one level: now the system's deployable pieces appear — the web application, the API service, the database, a message queue, a mobile app — each a separate box, with arrows showing how they communicate and over what protocol. This is the first level aimed squarely at a technical audience: an engineer joining the team can see the moving pieces and how they're wired together without yet caring what's inside any one of them. Note the terminology trap: "container" here means a deployable unit in the C4 sense, not specifically a Docker container, though a container in the Docker sense is usually also a container in the C4 sense.

**Component** zooms in once more, into a single container: what are the major structural building blocks inside this one service, and how do they collaborate? This is still above the level of individual classes — a component might be "the order validation module" or "the payment gateway adapter" — aimed at the engineers who work inside that specific container and need to understand its internal shape before changing it.

**Code** is the deepest level, and C4 deliberately spends the least effort here. Brown's own position is candid about this: a class diagram generated straight from the code (or just the code itself, read directly) documents this level more accurately than a hand-maintained one ever will, because a hand-drawn diagram starts drifting the moment the code changes and nobody has a strong incentive to keep updating it. C4 treats Code as optional and tool-generated rather than something an architect maintains by hand.

## Matching the diagram to the room

The real discipline C4 teaches isn't the specific boxes-and-arrows notation — it's a habit: before drawing anything, decide who's in the room and pick the one level that matches what they actually need to know. A Context diagram in a technical design review wastes the room's time restating what everyone already knows. A Component diagram in front of an executive sponsor loses them in internal structure they have no use for and no way to evaluate. Matching detail to audience is the actual communication skill; C4 just gives it four named, consistent stops instead of leaving the zoom level to each diagram's author to improvise every time.

## Beyond the diagram itself

Communicating architecture well goes past picking the right diagram level. The same trade-off reasoning recorded in an ADR (the previous lesson) needs to be narratable out loud in a way a non-architect can follow: leading with the business consequence of a decision — cost, risk, time to ship — before the technical mechanism behind it, because that's the order a stakeholder's own priorities run in. A Context diagram and a one-paragraph plain-language summary of "why we chose this" together cover most of what a non-technical audience actually needs from an architect; the Container and Component diagrams, and the full ADR, are there for the engineers who'll actually build and maintain the thing.
