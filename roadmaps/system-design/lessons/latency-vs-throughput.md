---
title: "Latency vs. Throughput"
description: Why latency and throughput trade off through batching, why averages hide the truth, and how one slow service can dominate a whole request.
roadmap: system-design
node: latency-vs-throughput
group: design-fundamentals-and-estimation
video:
  provider: youtube
  id: "3JdQOExKtUY"
  start: 0
  end: null
creator:
  name: Hussein Nasser
  channel_url: https://www.youtube.com/@hnasr
duration_seconds: 375
level: intermediate
prerequisites:
  - performance-vs-scalability
takeaways:
  - Latency is how long one request takes, end to end; throughput is how many requests the system completes per unit of time — they are different measurements, and a system can be good at one and bad at the other.
  - Batching requests together (writing 100 rows in one transaction instead of 100) raises throughput by amortizing fixed overhead, but it raises latency for the first item in the batch, which has to wait for the rest to arrive.
  - An average response time can look fine while most users have a terrible experience, because a handful of very slow outliers get smoothed away by a much larger number of fast ones.
  - Percentiles (p50, p95, p99) report the latency below which that percentage of requests fall, which is why p99 — the slowest 1% — is the number that reveals a tail-latency problem an average hides.
  - When one request fans out to call several backend services and waits for all of them, the overall latency is set by the slowest call, not the average call — so the probability that at least one of N calls is slow grows with N.
  - Tracking p95/p99 instead of average latency, and budgeting for fan-out amplification explicitly, is standard practice precisely because both failure modes are invisible until you look for them on purpose.
check_questions:
  - q: What is the practical difference between latency and throughput?
    a: Latency is the time a single request takes from start to finish, while throughput is the number of requests or units of work the system completes per unit of time, and a system can score well on one while scoring poorly on the other.
  - q: Why does batching improve throughput but hurt latency for individual items in the batch?
    a: Batching spreads a fixed per-operation cost (like a disk write or a network round trip) across many items at once, which raises the total completed per second, but the first items in a batch have to sit and wait for the batch to fill before any of them gets processed, which increases their individual wait time.
  - q: A service has an average latency of 50ms but a p99 of 4 seconds. What does that gap tell you, and why wouldn't the average alone have caught it?
    a: It tells you that roughly 1% of requests are dramatically slower than the rest, which is a real, user-visible tail-latency problem; the average alone wouldn't catch it because a small number of very slow outliers barely move an average computed over a much larger number of fast requests.
deeper:
  - label: "Latency is a distribution, not a number: p99, fan-out and the tail at scale"
    url: "https://www.youtube.com/watch?v=co2LZv0n4U4"
  - label: "Latency vs Throughput: The System Design Mistake Engineers Make"
    url: "https://www.youtube.com/watch?v=Wm9_0VqPft4"
related_formulas: []
---

## Two numbers, not one

Latency and throughput both describe "how fast" a system is, but they answer different questions, and conflating them leads to fixing the wrong problem.

**Latency** is the time one request takes, from the moment it's sent to the moment its response arrives. It's a duration, measured in milliseconds or seconds, and it's a property of a single request.

**Throughput** is the rate at which the system completes work, measured in requests per second, transactions per second, or megabytes per second. It's a property of the system under some amount of concurrent load, not of any one request.

A system can be excellent at one and poor at the other. A single, unshared SQLite file can answer one query in under a millisecond — superb latency — but can only sustain a modest number of writes per second because writes serialize against each other; its throughput ceiling is low. A big batch-processing pipeline might take ten minutes of latency per job, but by running thousands of jobs in parallel across a cluster, it can sustain enormous aggregate throughput. Neither system is strictly "fast" or "slow" — the right question is always "fast or slow at what."

## The batching tradeoff

The clearest place the two pull in opposite directions is batching. Say a service writes events to a database, and each write has a fixed overhead — say 5ms of network round-trip and commit cost — on top of the actual work.

- Writing one event at a time: every event pays the full 5ms overhead, so overhead dominates, and you might sustain only ~200 writes/second per connection — but each individual event is acknowledged in roughly 5ms.
- Batching 100 events per write: the 5ms overhead is now amortized across 100 events, so you might sustain ~10,000+ events/second of throughput — a huge win — but any individual event now waits for 99 others to accumulate before the batch fires, so its latency might jump to 50-200ms depending on how the batch window is tuned.

Neither configuration is "wrong." A background analytics pipeline that doesn't care when an individual event lands, only that the pipeline keeps up with total volume, should batch aggressively. A user-facing "like" button that needs to confirm instantly should not — it should accept the lower throughput-per-connection in exchange for low latency, and get its throughput some other way (more connections, more servers) instead of batching user-visible writes. This is the general shape of the tradeoff: anything that amortizes fixed cost across more units of work (batching, buffering, compression) tends to raise throughput at the cost of the latency experienced by the unit that has to wait its turn.

## Why averages lie

Suppose a service handles 1,000 requests, and 990 of them complete in 10ms while 10 of them — hitting a cold cache, a slow disk seek, or a garbage-collection pause — take 2,000ms. The average latency across all 1,000 requests is just over 29ms. That number looks healthy. But 1% of your users just waited two full seconds, and if your product has any kind of retry or page-load dependency on that call, that 1% might translate into a disproportionate amount of visible pain, abandoned sessions, or cascading retries that make things worse.

This is why system design practice reports **percentiles** instead of (or alongside) averages:

- **p50** (the median) is the latency below which half of all requests fall — a decent proxy for the "typical" experience.
- **p95** is the latency below which 95% of requests fall — the slowest 5% are worse than this.
- **p99** is the latency below which 99% of requests fall — the slowest 1% are worse than this, and it's specifically designed to surface the outliers an average smooths away.

A service with p50 = 10ms and p99 = 2,000ms has exactly the distribution described above: typically fast, with a real and measurable tail problem. Engineering teams set Service Level Objectives in terms of p95/p99 (covered later in this roadmap) precisely because "average latency is fine" is compatible with a genuinely bad experience for a meaningful slice of users.

## Tail latency amplification under fan-out

The tail gets worse, not better, once a single request depends on multiple backend calls — which is the normal shape of any non-trivial service. Suppose a page-load request fans out to 20 independent backend calls (a recommendations service, a user-profile service, an inventory check, and so on), and each individual call has a p99 of 100ms — meaning each call, on its own, is slow only 1% of the time.

If the page has to wait for *all 20* calls before it can respond, the probability that *at least one* of them is in its slow 1% isn't 1% — it's roughly `1 - (0.99)^20 ≈ 18%`. Nearly one in five page loads ends up waiting on some call's tail latency, even though any individual backend looks fine 99% of the time. Add more fanned-out calls and the amplification gets worse — at 50 calls, the odds that the page is dragged down by at least one straggler rise to roughly 40%. This is why a system composed of many small, individually-fast services can end up with surprisingly bad end-to-end tail latency: the tail compounds across the fan-out, and the overall latency is set by the slowest call in the set, not the typical one.

The practical responses — setting aggressive per-call timeouts, issuing redundant "hedged" requests to a backup replica when a call is running slow, and reducing unnecessary fan-out in the first place — come up again later in this roadmap under reliability patterns. The habit worth building now is to ask, for any fan-out design, "what's my p99 once I multiply this across every call this request makes?" rather than assuming each call's own tail is the whole story.
