---
title: "Cloud Design Patterns"
description: Valet key, gatekeeper, competing consumers, and pipes and filters — cloud patterns this site's other architecture lessons don't already cover.
roadmap: software-architect
node: cloud-design-patterns
group: cloud-and-enterprise-patterns
video:
  provider: youtube
  id: "TuZZIGSbFfQ"
  start: 0
  end: null
creator:
  name: NDC Conferences
  channel_url: https://www.youtube.com/@NDC
duration_seconds: 3808
level: intermediate
prerequisites: []
takeaways:
  - "The Valet Key pattern hands a client a time-limited, scope-limited token — like Azure's SAS or an S3 pre-signed URL — so it can upload or download a large file directly against storage, without the application server ever sitting in the middle of the data transfer."
  - "The Gatekeeper pattern puts a thin, hardened validation layer in front of the real application, so the component with the broadest attack surface (anything facing untrusted input) is also the one with the least privilege — it holds no database credentials or service keys at all."
  - "Competing Consumers lets several worker instances pull from the same queue at once; a slow or crashed worker doesn't stall anyone else, and throughput scales by adding workers rather than by making any one worker faster."
  - "Pipes and Filters breaks one monolithic processing step into a chain of small, independently deployable, independently scalable stages, each one reading from its predecessor's output and writing to its successor's input."
  - "All four patterns are really one idea applied at different seams of a system: narrow the interface and the privilege at a boundary down to exactly what that boundary needs, instead of handing out broad access or doing everything in one undifferentiated chunk."
check_questions:
  - q: Why does the Valet Key pattern have the client talk to storage directly instead of routing the file through the application server?
    a: Routing a large upload or download through the application server makes the server a bottleneck for pure data transfer it adds no value to; a time-limited, scoped token lets the client talk to storage directly, freeing the server to do the things only it can do.
  - q: What specifically does the Gatekeeper pattern reduce, and how?
    a: It reduces the blast radius of a compromised internet-facing component by giving it no credentials to anything valuable — the gatekeeper can only validate and forward requests, so compromising it doesn't hand an attacker direct access to a database or a storage account.
  - q: In the Competing Consumers pattern, what happens to in-flight ordering guarantees when you add more workers?
    a: Strict ordering gets harder to guarantee, because messages can finish processing out of the order they were enqueued in when different workers pick them up and run at different speeds — a trade-off against the throughput and resilience gains the pattern buys you.
deeper: []
related_formulas: []
---

## Patterns for the seams of a cloud system

An architect spends a lot of time at the seams between components — the place where one service hands work, data, or trust to another. `system-design` already covers this site's core reliability and resilience patterns (circuit breakers, retries, bulkheads, health checks), and its architecture-patterns-at-scale group covers CQRS, the strangler fig, sidecars, and backend-for-frontend. This lesson covers four more cloud-native patterns, from the catalogs Microsoft Azure and AWS both document, that solve a narrower set of seam problems: who gets direct access to a resource, who gets to stand between the internet and your application, how work gets distributed across workers, and how a single processing step gets broken into a pipeline.

## Valet Key: handing out a scoped, expiring ticket

Imagine an app that lets users upload large video files. The naive design routes every byte through the application server, which reads the upload stream and writes it to storage — the server does real work (authentication, validation) for maybe the first kilobyte, then becomes an expensive, bandwidth-bound proxy for the rest. The **Valet Key** pattern fixes this: the server authenticates the request, then issues a **time-limited, scope-limited token** — a Shared Access Signature on Azure Blob Storage, a pre-signed URL on S3 — that lets the client talk to the storage service directly. The server never touches the file's bytes. The name comes from a car valet key: it starts the engine and opens the door, but it doesn't open the trunk or the glove box. The token here does one thing (write to this one blob, for the next ten minutes) and nothing else — if it leaks, the damage is bounded by scope and time.

## Gatekeeper: a thin, untrusted-facing layer with nothing to lose

The **Gatekeeper** pattern addresses a different question: what sits between the open internet and the part of your system that actually has access to sensitive resources? A gatekeeper is a dedicated, deliberately minimal process whose only job is to validate and sanitize incoming requests — checking size limits, rejecting malformed input, enforcing authentication — before forwarding anything onward. Critically, the gatekeeper itself holds **no credentials** to the database, the storage account, or any downstream service; it only knows how to talk to a separate, trusted host that does. If an attacker finds a bug in the gatekeeper and compromises it, they've compromised a component that can't do anything to your data, because it was never handed the keys. This is the architectural version of "don't let the receptionist hold the vault combination."

## Competing Consumers: many workers, one queue

Once work is expressed as messages on a queue — a common shape once you're decoupling producers from processors — the **Competing Consumers** pattern is the default way to scale processing: run several consumer instances that all pull from the same queue, each one claiming and processing whatever message is next. No message is assigned to a worker in advance; workers simply compete for whatever's available. This buys two things at once. First, throughput scales roughly linearly with the number of consumers, since the queue is the only shared state and most message brokers are built to hand out work from one queue to many readers cheaply. Second, it's self-healing: if one consumer crashes mid-message, a competently designed queue (one with visibility timeouts or equivalent) makes that message available again for another consumer to pick up, rather than losing it. The cost is ordering — once multiple workers are racing through the same queue at different speeds, you give up any guarantee that messages finish in the order they arrived, and a system that genuinely needs strict ordering needs a different pattern (partitioned queues keyed so related messages always land on the same consumer, for instance).

## Pipes and Filters: a pipeline instead of one giant step

**Pipes and Filters** takes a single complex processing task — say, ingesting an uploaded document, extracting its text, running it through a classifier, and indexing the result — and splits it into a chain of independent **filters**, each one doing exactly one transformation, connected by **pipes** that carry data from one filter's output to the next filter's input. Each filter can be developed, deployed, scaled, and replaced independently: if classification is the slow, CPU-heavy stage, you scale out classifier instances without touching extraction or indexing at all. This is the same idea Unix shell pipelines popularized decades ago (`cat file | grep pattern | sort | uniq -c`), applied to distributed processing: small, composable, single-purpose stages beat one large function that does everything and can only be scaled, tested, or redeployed as a single indivisible unit.

## Why these four, and not the ones system-design already owns

It's worth being explicit about the boundary: if the problem you're facing is a flaky downstream call, a cascading failure, or routing requests to the right service version, those are `system-design`'s circuit-breakers-and-retries, bulkheads-and-graceful-degradation, and sidecar-ambassador-and-service-mesh lessons, and re-deriving them here would just be a shallower second copy. What Valet Key, Gatekeeper, Competing Consumers, and Pipes and Filters have in common instead is that they're about **shaping a boundary** — what a client is allowed to touch directly, what an untrusted-facing process is allowed to know, how work gets divided among equal peers, and how a single step gets decomposed into a chain. That's squarely an architect's decision, made once, that every component downstream of it has to live with.
