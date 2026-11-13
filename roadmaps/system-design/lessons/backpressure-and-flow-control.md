---
title: "Backpressure and Flow Control"
description: What happens when a producer outpaces a consumer, why bounded queues and load shedding are the real fix, and why a bigger queue usually isn't.
roadmap: system-design
node: backpressure-and-flow-control
group: async-systems-and-messaging
video:
  provider: youtube
  id: "Vwb_rliN_2k"
  start: 0
  end: null
creator:
  name: CodeLucky
  channel_url: https://www.youtube.com/@thecodelucky
duration_seconds: 295
level: intermediate
prerequisites:
  - message-queues-and-async-communication
  - pub-sub-and-event-driven-architecture
takeaways:
  - Backpressure is what happens when a producer generates work faster than a consumer can process it — without something to absorb or push back on that mismatch, the gap between the two just keeps growing.
  - A queue between producer and consumer absorbs a temporary burst, but an unbounded queue just delays the failure rather than preventing it — it keeps accepting work while quietly building a backlog and consuming more and more memory.
  - "A bounded queue forces the mismatch to surface immediately: once it's full, something has to give — the producer slows down, waits, or the new item gets rejected — instead of memory usage growing without limit."
  - Load shedding means deliberately dropping or rejecting some work when a system is overloaded, so that the requests it does accept can still be served at an acceptable speed, rather than accepting everything and serving all of it slowly or not at all.
  - Making the queue bigger usually doesn't fix a sustained rate mismatch — it only buys a longer runway before the backlog manifests as unbounded latency or an out-of-memory crash, and in the meantime the data sitting in the queue gets staler and staler.
  - Real flow control works by propagating the slowdown signal backward through the chain — a consumer telling its queue to stop accepting, that queue telling its producer to pause — so the system degrades predictably instead of failing all at once.
check_questions:
  - q: A producer writes 10,000 messages per second into a queue, but the consumer can only process 6,000 per second. If the queue has no size limit, what actually happens over the next hour, even though no error is ever thrown?
    a: The queue keeps accepting every message because nothing stops it, so the backlog grows by about 4,000 unprocessed messages every second; over an hour that's roughly 14.4 million messages sitting in memory, which either exhausts the machine's memory and crashes the process, or means a message now waits so long before being processed that it's effectively stale by the time it's handled.
  - q: Why does making the queue bigger usually fail to fix a sustained speed mismatch between a producer and a consumer, as opposed to a short burst?
    a: A bigger queue only adds more buffer space to delay the moment the backlog becomes visible — it doesn't change the fact that the consumer processes slower than the producer produces, so the backlog keeps growing regardless of the limit; a bigger queue helps with a short burst that the consumer can later catch up on, but it does nothing for a mismatch that's sustained, since the gap just keeps widening until the bigger limit is also hit.
  - q: What's the difference between bounding a queue and load shedding, and why might a system need both?
    a: Bounding a queue sets a hard limit on how much unprocessed work can pile up before new work is pushed back on or rejected; load shedding is the deliberate choice of which work to drop once the system is overloaded, so that what remains can still be served well — a system typically needs both, because a bounded queue tells you *when* you're overloaded, and load shedding (or backpressure to the producer) tells you *what to do about it*.
deeper: []
related_formulas: []
---

## The mismatch that backpressure describes

Any time one part of a system produces work faster than the part downstream of it can consume it, you have a speed mismatch. A web server accepting requests faster than a database can execute them. A sensor emitting readings faster than an analytics pipeline can ingest them. A publisher putting events on a topic faster than a subscriber can process them. **Backpressure** is the general name for what a system does — or fails to do — about that gap.

The gap itself isn't optional to deal with; it's physics. If the producer is faster than the consumer, the difference between them has to go *somewhere*: it either accumulates as a growing backlog, or it gets rejected, or the producer gets slowed down to match. Which of those three happens is a design decision, and "none of the above" isn't actually one of the choices — it just means the backlog option happens by default, usually in a form nobody explicitly planned for.

## Why an unbounded queue only delays the problem

A queue between a producer and consumer is the standard way to absorb a *temporary* mismatch — a short burst where the producer is briefly faster, followed by a lull where the consumer catches up. That's exactly what queues are good at, and it's why the message-queues lesson described them as smoothing load.

The trap is assuming that if a queue helps with a burst, an unlimited queue must help even more. It doesn't — it just hides a *sustained* mismatch instead of a temporary one. If the consumer is reliably slower than the producer, an unbounded queue keeps accepting everything, and the backlog grows without any limit at all:

```
Producer rate:  10,000 msg/sec
Consumer rate:   6,000 msg/sec
Net backlog growth: 4,000 msg/sec, forever, until something changes

After 1 minute:  240,000 messages queued
After 1 hour:  14,400,000 messages queued
```

Nothing has crashed yet in that picture — which is precisely the problem. The system looks fine from the producer's side (every write succeeds) right up until memory runs out, or until a message finally gets processed so many minutes or hours late that it's no longer useful. A bigger queue buys a longer runway before either of those happens; it doesn't change the underlying rate mismatch that's causing the backlog to grow in the first place. That's why "just make the queue bigger" is usually the wrong fix for a sustained mismatch — it treats the symptom (the queue filled up) rather than the cause (the consumer is, and remains, slower than the producer).

## Bounding the queue forces the decision to happen now

A **bounded queue** — one with a fixed maximum size — doesn't solve the rate mismatch either, but it does something more useful: it forces the system to make an explicit decision about what happens once the buffer is full, right now, instead of silently accumulating an invisible backlog that turns into a crash later.

Once a bounded queue is full, there are really only a few honest options:

- **Push back on the producer** — block it, or have it explicitly wait, until there's room. This is real backpressure: the signal that "slow down" propagates backward from the consumer, through the queue, to whatever's producing the work.
- **Reject the new item** outright, and let the producer (or its caller) decide what to do — retry later, fail the request, fall back to something cheaper.
- **Drop something** — either the new arrival or, in some designs, the oldest item already waiting — to make room.

Any of these surfaces the problem immediately, as an explicit, visible event, rather than as a slow memory leak that only becomes an incident hours later.

## Load shedding: choosing what not to do

**Load shedding** is the deliberate version of "reject the new item": when a system is overloaded, it actively refuses some fraction of incoming work so that the work it *does* accept can still be handled at an acceptable speed. This sounds counterintuitive — rejecting requests on purpose — but the alternative is usually worse: a system that tries to accept everything under overload typically ends up serving *everything* slowly, including the requests that arrived before the overload started, which often causes timeouts and retries that make the overload even worse.

A rate limiter rejecting excess requests before they ever reach an overloaded service, or a queue dropping low-priority messages first while keeping high-priority ones, are both load shedding in practice. The point isn't to never fail — it's to fail in a controlled, chosen way, for a bounded set of requests, instead of failing unpredictably for all of them once the system tips over.

## Why propagating the signal backward matters

The common thread across bounded queues, backpressure, and load shedding is that each one makes the slowdown *visible and actionable* at the point where it happens, instead of letting it hide as an ever-growing backlog somewhere downstream. A system with real flow control degrades predictably under overload — some requests get rejected or delayed, clearly and on purpose — rather than appearing healthy right up until it runs out of memory or every request becomes too late to matter.
