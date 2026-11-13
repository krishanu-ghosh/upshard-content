---
title: "Performance vs. Scalability"
description: Why a fast system isn't automatically a scalable one, and the vertical-ceiling-vs-horizontal-growth framing that explains the difference.
roadmap: system-design
node: performance-vs-scalability
group: design-fundamentals-and-estimation
video:
  provider: youtube
  id: "xpDnVSmNFX0"
  start: 0
  end: null
creator:
  name: Gaurav Sen
  channel_url: https://www.youtube.com/@gkcs
duration_seconds: 476
level: intermediate
prerequisites:
  - what-is-system-design
takeaways:
  - Performance measures how fast a system handles a single request or a small, fixed load; scalability measures whether that speed holds up as the load grows.
  - A system can be fast-but-not-scalable — snappy for one user but it falls over past some load — or scalable-but-slow — stays functional at huge load, but every individual request is sluggish.
  - Improving performance (faster code, better indexes, a local cache) often does nothing for scalability, because it doesn't change what happens when load multiplies past the point that single machine can absorb.
  - Vertical scaling (a bigger machine) buys performance headroom quickly but hits a hard ceiling — there's a biggest machine you can buy, and it costs disproportionately more as you approach it.
  - Horizontal scaling (more machines sharing the load) has a much higher ceiling but requires the system to be built so that adding a machine actually helps, which isn't automatic.
  - Confusing the two leads to the wrong fix — throwing a faster server at a system that's actually falling over under concurrent load, or splitting across ten machines to solve a problem that was really just one slow, unindexed query.
check_questions:
  - q: What's the difference between a performance problem and a scalability problem?
    a: A performance problem means the system is slow even under light, single-user load; a scalability problem means the system is fine under light load but degrades or fails as load increases, even though nothing about a single request changed.
  - q: What does it look like when a system is fast for one user but not scalable?
    a: A system with no caching and a single database connection can answer one request in a few milliseconds, but the moment concurrent requests pile up, connections queue, latency climbs, and the system falls over well before it's handling meaningful traffic.
  - q: Why does vertical scaling eventually stop being a viable answer to a scalability problem?
    a: Because it depends on a single machine getting bigger, and there's a largest machine available at any point in time — cost also rises faster than capacity as you approach the top of the hardware range, so it becomes both physically and economically limited.
deeper: []
related_formulas: []
---

## Two different questions

"Is this system fast?" and "will this system stay fast as more people use it?" sound similar but are answered by completely different kinds of engineering, which is why system design separates them into two named concepts.

**Performance** is about a single unit of work: how long does one request take, how much memory does one operation use, how fast does one query return. You improve performance by making the work itself cheaper — a better algorithm, an index on the right column, less serialization overhead, a warmer cache. Performance is measurable by hitting the system once, or a few times, and timing it.

**Scalability** is about what happens as the *amount* of work grows: does the system keep meeting its latency and correctness requirements as concurrent users, data volume, or request rate increases, or does it degrade — or break outright — past some point? Scalability isn't visible in a single request's timing at all; it only shows up when you push load and watch what happens to that timing as load keeps rising.

## Two failure modes that look similar but aren't

Because performance and scalability are different axes, a system can fail on either one independently:

**Fast but not scalable.** A single-process web server backed by one database connection can answer a lone request in 5 milliseconds — genuinely fast. Send it 500 concurrent requests, though, and requests start queuing behind that one connection; latency for the 500th request might be seconds, not milliseconds, and the system may start timing out or crashing well before "the whole internet" is hitting it. Nothing about how fast *one* request runs changed; what changed is that the system has no way to do many of them at once. This is the classic shape of a scalability problem: the system was never designed to add capacity when load grows, so it has a ceiling that gets hit quickly.

**Scalable but slow.** The opposite also happens: a system built from the start to run across many machines, with load balancing and horizontal partitioning, can hold up fine under enormous concurrent load — it doesn't fall over — while still being unpleasantly slow for any individual user, because each request has to hop through several services, wait on a few network calls, and get re-assembled before a response goes out. It scales (adding more load doesn't break it), but it's never been optimized to be *fast*. A batch analytics pipeline that reliably processes a billion events a day, but takes 40 seconds to answer any single ad-hoc query, is scalable-but-slow by design — that's an acceptable tradeoff for that workload, but it would be a bad fit for, say, an autocomplete box.

The practical consequence: don't diagnose a scalability problem by making the slow path faster, and don't diagnose a performance problem by adding more servers. A slow single query gets fixed by an index, not a second datacenter. A system that falls over at 10x today's traffic gets fixed by changing how load is distributed, not by shaving milliseconds off a function that was never the bottleneck.

## The ceiling vs. the floor you can keep raising

There are two broad ways to give a system more capacity, and they trade off very differently.

**Vertical scaling** means making the existing machine bigger: more CPU cores, more RAM, a faster disk. It's the simplest possible move — nothing about the application's architecture has to change — and it genuinely buys headroom, often enough to outrun months or years of growth for a smaller system. But it has a hard ceiling: there is a biggest machine that exists to buy at any point in time, and the price curve isn't linear — the jump from a very large machine to the largest available one tends to cost disproportionately more per unit of extra capacity. Vertical scaling also leaves you with a single point of failure: one machine, however large, is still one machine.

**Horizontal scaling** means adding more machines and spreading load across them. The ceiling is much higher in principle — you can generally buy another server — but it isn't free: the application has to be built so that adding a machine actually increases usable capacity, which usually means no single machine can hold state that the others don't know about, requests need to be distributed sensibly across the fleet, and anything shared (a database, a cache) has to be able to handle traffic from many machines at once rather than one.

This lesson only needs the distinction, not the mechanics — load balancing, sharding, replication, and the operational cost of running a fleet instead of a box are each covered later in this roadmap, where the "how" of horizontal scaling gets the depth it deserves. What matters here is the framing: performance and scalability are different problems with different symptoms, and vertical vs. horizontal scaling are different tools with a different ceiling, and mixing up either pair sends you fixing the wrong thing.
