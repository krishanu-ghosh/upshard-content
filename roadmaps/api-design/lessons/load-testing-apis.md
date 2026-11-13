---
title: "Load Testing APIs"
description: Load testing an API specifically — a realistic traffic pattern, the real difference between load, stress, spike, and soak tests, and what to assert against.
roadmap: api-design
node: load-testing-apis
group: documentation-testing-and-quality
video:
  provider: youtube
  id: "r-Jte8Y8zag"
  start: 0
  end: null
creator:
  name: Nick Chapsas
  channel_url: https://www.youtube.com/@nickchapsas
duration_seconds: 860
level: intermediate
prerequisites:
  - mocking-and-virtualizing-apis
takeaways:
  - "\"Hit it as hard as possible\" isn't a load test plan — a realistic test models actual concurrency (how many simultaneous users or callers you expect, not total requests fired as fast as the tool can send them) and a realistic request mix (the real ratio of reads to writes, cheap endpoints to expensive ones), because a flat barrage of identical requests exercises a traffic shape your API will never actually see in production."
  - "Load, stress, spike, and soak tests ask four different questions: load testing checks behavior at expected peak traffic; stress testing pushes past that peak to find where and how the system actually breaks; spike testing checks recovery from a sudden, short burst rather than a gradual ramp; soak testing runs a sustained, moderate load for hours to catch what only shows up over time — memory leaks, connection pool exhaustion, slow resource leaks a short test never lives long enough to reveal."
  - "The right thing to assert against is latency percentiles (p50, p95, p99) and the error rate under load, not an average response time — an average of 80ms hides a p99 of 4 seconds that a real fraction of your users are actually experiencing, which is the whole reason percentiles are the number that matters, not the mean."
  - "A passing load test and a system that's actually ready for production traffic aren't the same claim — the test is only as realistic as the traffic pattern, request mix, and data volume it modeled, so a test run against a mostly-empty database or a traffic shape nothing like real usage can pass cleanly and still miss the way the system actually fails in production."
  - "Load testing is a pre-production gate you run before a launch or a major change, not an ongoing practice — `api-performance-metrics-and-monitoring`, later in this roadmap, owns watching the same latency and error-rate signals continuously once the system is live and real traffic, not a simulated pattern, is hitting it."
check_questions:
  - q: Why is "fire as many requests as possible for as long as possible" a bad load test design?
    a: Because it tests a traffic shape no real system actually experiences — production traffic has a specific concurrency level (real simultaneous users, not infinite parallel requests) and a specific mix of cheap and expensive, read and write endpoints, and a flat barrage of identical requests at maximum throughput tells you how the system behaves under an unrealistic shape of load, not whether it'll hold up under the traffic pattern it will actually see.
  - q: A team's load test shows an average response time of 90ms and declares the API healthy. What might that number be hiding, and what should they have looked at instead?
    a: An average can be pulled down by a large number of fast, cheap requests while hiding a meaningful tail of slow ones — a p99 of several seconds is entirely compatible with a 90ms average if most requests are fast and a smaller fraction are very slow. They should assert against latency percentiles (p50/p95/p99) and the error rate under load, since those actually show what a real user at the slow end of the distribution experiences, which the average erases.
  - q: What's the difference between a stress test and a soak test, and what does each one catch that the other doesn't?
    a: A stress test pushes load past the system's expected peak to find the breaking point and how it fails there — at what concurrency does latency blow up or errors start, and does it fail gracefully or catastrophically. A soak test keeps load at a sustained, moderate (not extreme) level for a long duration — hours, not minutes — to catch problems that only emerge over time, like a slow memory leak, a connection pool that never releases connections, or a cache that grows unbounded, none of which a short stress test at any load level would last long enough to reveal.
deeper:
  - label: "How to do Performance Testing with k6"
    url: "https://www.youtube.com/watch?v=ghuo8m7AXEM"
related_formulas: []
---

## Load testing an API is not a stress contest

It's tempting to treat load testing as a single question — "how many requests per second can this API handle before it falls over?" — and answer it by pointing a tool at an endpoint and cranking the throughput as high as it'll go. That answers a real question, but it's the wrong first question, and treating it as the only one produces a test that doesn't look anything like the traffic your API actually faces.

A realistic load test starts from two decisions: **concurrency** — how many requests are genuinely happening at the same time, modeled on how many real users or calling services you expect, not how fast a single test runner can fire requests sequentially — and **request mix** — the real-world ratio between endpoints. A typical API sees far more reads than writes, and within reads, far more calls to cheap, cacheable lookups than to expensive aggregation endpoints. A load test that sends an even split of traffic across every endpoint, or worse, hammers a single endpoint at max throughput, is testing a shape of load that doesn't resemble anything a real deployment will see, and a pass on that test tells you very little about what happens Tuesday at 9am when real traffic shows up.

## Four tests, four different questions

"Load testing" is often used loosely to mean any of four distinct tests, each answering a different question about the same system:

- **Load testing** proper checks behavior at your expected peak — the traffic level you've actually sized the system for. Does latency stay acceptable, do error rates stay near zero, at the load you're planning to actually handle?
- **Stress testing** deliberately goes past that peak, ramping concurrency up until something gives, to find where the breaking point actually is and, just as importantly, how the system fails there — does latency degrade gradually and predictably, or does the whole system fall over at once past some threshold? A system that degrades gracefully under 3x expected load is in a very different risk position than one that falls off a cliff at 1.2x.
- **Spike testing** checks a different shape entirely: a sudden, short burst of traffic rather than a gradual ramp — the Black Friday flash sale or the notification push that sends everyone to the app at once. A system can handle a slow ramp to a given concurrency perfectly well and still fall over on a spike to the same concurrency, because autoscaling, connection pools, and caches that warm up gradually under a ramp get no warning under a spike.
- **Soak testing** runs a sustained, moderate load — not extreme, just real — for hours rather than minutes. This is the one that catches slow failure modes a short test can't: a memory leak that's invisible at the 5-minute mark and obvious at the 5-hour mark, a connection pool that leaks a connection per thousand requests, a cache that grows without eviction until it exhausts available memory.

None of these substitutes for the others — a system can pass a load test and a stress test cleanly and still fail a soak test eight hours in, which is exactly the failure mode the first two are structurally incapable of catching.

## What to actually assert against

The metric that matters is not the average response time. An average hides exactly the information you need: if 95% of requests return in 40ms and 5% take 3 seconds, the average can land around 180ms — a number that looks perfectly healthy while a real, meaningful slice of your users is having a bad time. This is why the standard practice is asserting against **latency percentiles** — p50 (the median, what a typical request experiences), p95, and p99 (the tail, what your slowest real users experience) — rather than a single mean. A load test that only checks "average latency under 200ms" can pass while p99 sits at several seconds, and that gap is precisely where real user complaints come from.

Alongside latency percentiles, assert against **error rate under load** — not just whether the system stays up, but whether the fraction of failed or timed-out requests stays within an acceptable bound as load increases. A system that keeps responding but starts failing 8% of requests at peak load hasn't "survived" the test in any useful sense, even if nothing crashed.

## The limits of a passing test, and where this leads next

A load test is only as good as the traffic pattern and data it's built against — hit a realistically-shaped traffic pattern against a mostly empty test database, or one seeded without the query patterns real production data creates (a user with ten thousand records instead of ten), and the test can pass cleanly while hiding a problem that only shows up at real data scale. `mocking-and-virtualizing-apis`, the lesson before this one, is useful for isolating dependencies during development, but a load test needs something closer to the real system, with real data volume, to tell you anything trustworthy about production readiness.

Load testing is also explicitly a pre-production gate, not something you run once and move on from — you run it before a launch, before a major architectural change, before a traffic-driving campaign, to answer "is this ready for the load we expect." `api-performance-metrics-and-monitoring`, later in this roadmap's security-performance-and-compliance group, picks up the same two signals — latency percentiles and error rate — but for the ongoing, different problem of watching real production traffic continuously after launch, not a simulated pattern run ahead of time.
