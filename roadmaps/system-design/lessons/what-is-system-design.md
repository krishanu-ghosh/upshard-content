---
title: "What Is System Design?"
description: System design as a discipline — functional vs. non-functional requirements, and how to work through an open-ended design problem without guessing.
roadmap: system-design
node: what-is-system-design
group: design-fundamentals-and-estimation
video:
  provider: youtube
  id: "SqcXvc3ZmRU"
  start: 0
  end: null
creator:
  name: Gaurav Sen
  channel_url: https://www.youtube.com/@gkcs
duration_seconds: 561
level: intermediate
prerequisites: []
takeaways:
  - System design is the practice of deciding how a software system's parts fit together so it meets both its functional requirements (what it does) and its non-functional requirements (how well it does it under real load).
  - A system design problem is deliberately open-ended — there is no single correct diagram, only choices that are well-justified or poorly-justified given a stated set of constraints.
  - The right first move is almost always to narrow scope — pin down the use cases that matter, the rough scale, and which non-functional requirements take priority, before drawing a single box.
  - Starting with the simplest design that could plausibly work, and then scaling only the part that actually breaks under load, beats guessing at a "final" architecture up front.
  - A good design conversation is a dialogue where assumptions are stated out loud and tradeoffs are named explicitly, not a monologue reciting a memorized architecture.
  - Every design decision in this roadmap — caching, sharding, consensus, queues — is really a non-functional-requirement tradeoff in disguise, which is why this lesson's vocabulary recurs everywhere else.
check_questions:
  - q: What is the difference between a functional and a non-functional requirement in system design?
    a: A functional requirement describes a specific behavior the system must perform, such as letting a user post a comment, while a non-functional requirement describes a quality the system must have while doing it, such as handling 10,000 comments per second or staying available during a server failure.
  - q: Why is clarifying scope and constraints usually the first step in a system design conversation, rather than drawing an architecture diagram?
    a: Because the right architecture depends entirely on the scale, consistency needs, and priority requirements involved, and guessing at those leads to over-building for a problem that doesn't exist or under-building for one that does.
  - q: What does "start simple, then scale deliberately" mean in practice?
    a: It means designing the smallest system that satisfies the functional requirements first, identifying which specific part fails to meet a non-functional requirement under realistic load, and only then adding the complexity (a cache, a second server, a queue) that fixes that specific bottleneck.
deeper: []
related_formulas: []
---

## A discipline, not a diagram

"System design" gets used loosely to mean the boxes-and-arrows diagram people draw in an interview, but the diagram is the output, not the discipline. System design is the practice of deciding how a software system's components — services, databases, caches, queues, load balancers — fit together so that the system does what it's supposed to do, and keeps doing it as the number of users, the amount of data, or the rate of requests grows.

That split between "does what it's supposed to do" and "keeps doing it under load" is the first vocabulary this roadmap needs:

- **Functional requirements** describe *behavior*: a user can upload a photo, a search returns matching results, an order gets charged once. These are the things you'd write a unit test for.
- **Non-functional requirements** describe *qualities* of that behavior: how many uploads per second the system must sustain, how long a search is allowed to take, how the system behaves when a server dies mid-charge. These are the things that determine the architecture.

Two systems can satisfy the exact same functional requirements with wildly different architectures, because they're built against different non-functional requirements. A photo-upload feature for a 50-person internal tool and a photo-upload feature for a billion-user social app do the same *thing*; they need completely different *systems* to do it, because the non-functional requirements — throughput, latency, durability, cost — are different by orders of magnitude. Everything later in this roadmap (sharding a database, adding a cache, choosing a consensus protocol) is a response to some specific non-functional requirement that a simple design can't satisfy anymore.

## Working an open-ended problem

A system design problem rarely comes with the numbers you need already attached. "Design a URL shortener" or "design a chat system" says nothing, on its own, about scale, consistency requirements, or budget. The discipline is in how you fill that in without guessing blindly:

1. **Clarify scope.** Decide which use cases are actually in play. A URL shortener that only needs to shorten and redirect is a much smaller problem than one that also needs click analytics, custom aliases, and link expiration. Cutting scope early is not cheating — it's the only way to design something concrete instead of something vague.
2. **Pin down constraints.** Get rough numbers: how many users, how many requests per second, how much data, how stale can a read be, how much downtime is tolerable. You don't need exact figures (the next lesson in this group covers estimating them), but you need enough to know whether you're building for a thousand requests a day or a hundred thousand a second — those produce different designs.
3. **Start simple.** Sketch the smallest design that satisfies the functional requirements: one application server, one database, synchronous calls. Resist the urge to add a cache, a queue, or a second region before you've identified that you need one.
4. **Identify the bottleneck.** Ask which non-functional requirement the simple design actually fails to meet. Maybe the database can't take the write volume. Maybe a single server can't hold all the traffic. Maybe a slow downstream call is blowing the latency budget. Name the specific failure.
5. **Scale deliberately.** Fix *that* bottleneck with the smallest change that addresses it — a cache in front of a hot read path, a queue to absorb a write spike, a second replica for availability — rather than redesigning everything. Then repeat: the fix often exposes the next bottleneck.

This loop — simple design, find the breaking point, fix it, repeat — is the actual engine behind most of the "add a cache," "add a load balancer," "add a queue" moves that make up the rest of this roadmap. None of those techniques is a default; each is the answer to a specific, identified constraint.

## What a good design conversation looks like

Whether it's a design-interview whiteboard session or a real design-review meeting before a project starts, a good system design conversation has a recognizable shape, and it isn't "recite an architecture from memory."

- **Assumptions are spoken, not assumed.** "I'm assuming reads outnumber writes by a lot here, since most users browse more than they post — let me know if that's wrong" invites correction before an hour gets spent on the wrong design.
- **Tradeoffs are named, not hidden.** Choosing a message queue over a direct synchronous call isn't free — it adds latency and operational complexity in exchange for resilience to spikes and decoupling. Saying that out loud, and explaining *why* it's the right trade for this problem, is the actual skill being evaluated (or the actual value being delivered in a real review).
- **The design can change under new information.** If a reviewer or interviewer says "actually, this needs to support 10x the write volume you assumed," a good conversation updates the design — swap a database, add sharding, rethink the bottleneck — instead of defending the original sketch.
- **Simplicity is defended, not apologized for.** A design that's "just a load balancer, two app servers, and a database" is often the *correct* answer for the stated scale, and justifying why you didn't add Kafka and twelve microservices is as much a sign of judgment as knowing when you would.

The video for this lesson walks through this process end to end — starting from an ambiguous prompt and narrowing it into a concrete design through exactly this kind of questioning. Watching how the questions get asked, in order, is more useful here than memorizing any particular answer, because the questions are what transfers to the next, different problem.
