---
title: "API Performance Metrics and Monitoring"
description: What to actually watch about an API's own contract in production — latency percentiles, per-endpoint error rate, throughput, and usage analytics.
roadmap: api-design
node: api-performance-metrics-and-monitoring
group: security-performance-and-compliance
video:
  provider: youtube
  id: "Y8FsB2wryjQ"
  start: 0
  end: null
creator:
  name: Hitesh Choudhary
  channel_url: https://www.youtube.com/@HiteshCodeLab
duration_seconds: 782
level: intermediate
prerequisites:
  - owasp-api-security-top-10
takeaways:
  - "An average latency number is nearly useless for an API with any real traffic — it's dominated by the fast, common case and hides the slow tail that's actually hurting users; p50, p95, and p99 tracked separately per endpoint are what a team should actually watch."
  - "p99 at 2 seconds on an endpoint averaging 80ms doesn't mean \"mostly fine, one weird outlier\" — at meaningful volume it means one in every hundred requests waits two full seconds, which is a lot of real, frustrated users if that endpoint gets called often."
  - "Error rate is only informative when it's sliced by endpoint and status code, not reported as one global number — a healthy-looking 0.3% overall error rate can hide one endpoint failing 40% of the time for a small but important slice of traffic, like refund processing for one payment provider."
  - "Throughput (requests per second, by endpoint) tells you which parts of your contract actually carry load, which is the input that should drive every scaling and caching decision — optimizing an endpoint nobody calls is wasted effort regardless of how slow it looks in isolation."
  - "API-specific usage analytics — which clients call which endpoints, and crucially, who's still on a deprecated version — close the loop with `api-versioning-strategies`: you cannot safely sunset an old version you haven't instrumented, because you have no way to know whether turning it off tomorrow is safe or an outage."
  - "This lesson is about what to watch, not how the watching gets built — the collection pipeline (structured logging, metrics scraping, trace propagation across services) belongs to `system-design`'s `monitoring-logging-and-alerting` and `distributed-tracing`, which this lesson deliberately doesn't repeat."
check_questions:
  - q: Why is average latency a worse number to put on a dashboard than p95 or p99, even though it's easier to compute?
    a: An average is pulled toward the bulk of fast, typical requests and can look perfectly healthy even while a meaningful fraction of requests are badly slow; percentiles report the actual experience at specific points in the distribution, so p99 directly answers "how bad is it for the worst 1% of requests," which is the question that correlates with real user complaints.
  - q: A team reports "our error rate is 0.4%, we're fine." What's missing from that claim?
    a: Which endpoint, and which clients. A global error rate averages across every endpoint and caller, so a single critical endpoint failing at a high rate for a small slice of traffic — or for one important enterprise client — can sit invisibly inside an overall number that looks acceptable; error rate needs to be sliced by endpoint and status code to be actionable.
  - q: Why does deciding to sunset an old API version depend on monitoring, not just on the deprecation announcement?
    a: An announcement tells clients you intend to retire the version; it doesn't tell you whether they actually migrated. Without usage analytics on that version's traffic, turning it off is a guess — it could be silent and safe, or it could break every client still calling it, and there's no way to know which without having instrumented who's still hitting it.
  - q: Why doesn't this lesson cover how traces or structured logs get collected and correlated across services?
    a: That's the observability pipeline's job, not the API contract's — `system-design`'s `monitoring-logging-and-alerting` and `distributed-tracing` already own how telemetry gets collected, shipped, and correlated across a distributed system. This lesson is scoped one layer up — given that pipeline exists, what should an API team specifically watch about the health of their own contract.
deeper:
  - label: "Percentile Tail Latency Explained (95%, 99%) — Monitor Backend performance with this metric"
    url: "https://www.youtube.com/watch?v=3JdQOExKtUY"
related_formulas: []
---

## Watching the contract, not the infrastructure

Once an API is live, "is it working" stops being a yes/no question and becomes a set of numbers a team has to decide to watch. This lesson is specifically about what those numbers are for an API's *own contract* — the thing a client actually experiences when it calls your endpoints — not the machinery that collects them. `system-design`'s `monitoring-logging-and-alerting` covers how logs, metrics, and traces get gathered in the first place, and `distributed-tracing` covers following one request across services; both are assumed here. The question this lesson answers is narrower and more specific to API design: given that pipeline, what should an API team actually be looking at about the health of the contract it ships.

## Percentiles, not averages

The first and most common mistake is watching a single average latency number. Averages are dominated by the common, fast case — if 95% of requests to `/orders/{id}` take 40ms and 5% take 3 seconds, the average still looks like "a bit over 100ms," which reads as healthy on a dashboard while one in twenty real users is sitting through a three-second wait. The fix is tracking percentiles instead: **p50** (the median — what a typical request experiences), **p95**, and **p99**, each watched per endpoint, not blended across the whole API. p99 answers a specific, useful question: how bad is it for the worst 1% of requests? At low traffic that 1% might be a rounding error; at real production volume — a few hundred requests per second on a popular endpoint — p99 at two seconds means thousands of genuinely bad experiences every hour, invisible to anyone only watching the average.

A second-order point worth internalizing: percentiles don't compose across endpoints the way averages tempt you to think they do. An API-wide p99 is a meaningless blend of a fast endpoint's tail and a slow endpoint's typical case. The number that's actually actionable is p99 *per endpoint*, because that's the granularity at which you can act — optimize a slow query, add a cache, or split out a heavy endpoint — without touching the rest of the API.

## Error rate, sliced the same way

Error rate suffers from the identical problem as latency: a single global number hides exactly the thing you need to see. An overall error rate of 0.4% sounds fine until you break it down by endpoint and status code and discover that `POST /payments/refund` is failing 35% of the time for requests from one specific upstream integration, while every other endpoint runs clean — a number that would never surface in a blended total but represents a genuinely broken feature for real customers. The practical discipline is tracking error rate per endpoint and per status-code class (4xx vs 5xx tell very different stories — a spike in 429s is your own rate limiting doing its job, a spike in 500s is your code breaking), and watching for divergence between an endpoint's error rate and its historical baseline rather than against one fixed global threshold.

## Throughput and who's actually calling

**Throughput** — requests per second, again broken down per endpoint — is what should drive capacity and caching decisions, because it tells you where the load actually is. An endpoint with terrible p99 latency that gets called twice a day is a much lower priority than a merely-mediocre endpoint that carries 80% of total traffic. This is also where API-specific usage analytics earn their keep in a way generic infrastructure metrics don't: which *clients* are driving load, not just which endpoints. A single misbehaving integration partner hammering an endpoint without backoff looks identical to organic growth in raw throughput numbers unless you can attribute traffic to the caller, usually via API key or client ID.

## The versioning tie-in

The single most concrete payoff of API usage analytics is deprecated-version tracking. `api-versioning-strategies` covers the `Deprecation` and `Sunset` headers as the signaling mechanism for retiring an old API version, and makes the point explicitly: you should never sunset a version you haven't instrumented. Usage analytics is what closes that loop — without a count of live traffic still hitting `/v1/orders`, "can we turn this off" is a guess, not a decision. With it, the decision becomes mechanical: zero traffic for a sustained window means it's safe; nonzero traffic means you know exactly which client to chase down before you flip the switch, instead of finding out from an outage report.
