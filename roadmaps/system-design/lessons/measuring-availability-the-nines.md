---
title: "Measuring Availability: The Nines"
description: What 99.9% vs 99.99% uptime means in real downtime, how availability multiplies across a dependency chain, and SLA vs SLO.
roadmap: system-design
node: measuring-availability-the-nines
group: consistency-availability-and-consensus
video:
  provider: youtube
  id: "hYu13kBenjE"
  start: 0
  end: null
creator:
  name: Twitter University
  channel_url: https://www.youtube.com/@TwitterUniversity
duration_seconds: 1910
level: intermediate
prerequisites:
  - availability-patterns-and-failover
takeaways:
  - "99.9% availability (three nines) allows about 8.76 hours of downtime a year; 99.99% (four nines) allows about 52.6 minutes — each extra nine cuts the allowed downtime by roughly 10x."
  - Availability numbers are deceptively easy to state and expensive to buy — going from three nines to four nines is a much bigger engineering lift than the small-looking jump from 99.9% to 99.99% suggests.
  - When a request depends on several services in series, the chain's overall availability is the product of each service's availability, which is always lower than the weakest link alone.
  - Adding a redundant, independent parallel path raises availability, because the whole path only fails when every parallel option fails at the same time.
  - An SLA is an externally-facing, often contractual promise to a customer; an SLO is the internal target a team actually engineers and operates against, usually set stricter than the SLA to leave a safety margin.
  - Treating "99.99% available" as a single number for an entire product hides where the risk actually lives — the weakest dependency in the chain usually caps what the whole system can promise.
check_questions:
  - q: Roughly how much more downtime per year does a 99.9% available service have compared to a 99.99% available service?
    a: About ten times as much — roughly 8.76 hours a year at 99.9% versus roughly 52.6 minutes a year at 99.99%.
  - q: If a request has to succeed through three independent services each with 99.9% availability, is the combined availability of the whole chain higher or lower than 99.9%?
    a: Lower — chaining dependencies in series multiplies their availabilities together (0.999 × 0.999 × 0.999 ≈ 99.7%), so the combined chain is always less available than its weakest single link.
  - q: What's the practical difference between an SLA and an SLO for the same service?
    a: The SLA is the external, often contractual availability promise made to customers, while the SLO is the internal target engineering actually holds itself to, typically set tighter than the SLA so there's margin before a breach of the customer-facing promise.
deeper:
  - label: "The Tyranny of the SLA"
    url: "https://www.youtube.com/watch?v=4cPqLuIXBnw"
related_formulas: []
---

## A percentage that hides a clock

"99.9% available" sounds almost perfect until it's converted into a clock. Availability is usually expressed as a percentage of time a system is up over some period, and the fastest way to make it concrete is to multiply the *downtime* side by the length of a year (8,760 hours) or a month (730 hours):

| Availability | Nickname | Downtime / year | Downtime / month |
|---|---|---|---|
| 99% | "two nines" | ~3.65 days | ~7.3 hours |
| 99.9% | "three nines" | ~8.76 hours | ~43.8 minutes |
| 99.99% | "four nines" | ~52.6 minutes | ~4.38 minutes |
| 99.999% | "five nines" | ~5.26 minutes | ~26.3 seconds |

The pattern worth internalizing: each additional nine doesn't shave a little off the downtime budget — it divides it by roughly 10. The jump from 99.9% to 99.99% looks like a small decimal change on paper but is actually a 10x tightening of the error budget, which is why it's also roughly a 10x jump in engineering effort: automated failover, multi-region redundancy, and real on-call process tend to be what separates a three-nines system from a four-nines one, not a tuning knob.

## Why a dependency chain is worse than its weakest link

A request rarely touches just one service. If a single API call has to pass through a load balancer, an auth service, an application server, and a database — four components, each independently available 99.9% of the time — the chain's overall availability isn't 99.9%. It's the *product* of all four:

```
0.999 × 0.999 × 0.999 × 0.999 ≈ 0.996 → 99.6% combined availability
```

That's roughly 35 hours of downtime a year for the chain, versus 8.76 hours for any one component alone, purely because every one of the four has to be up at the same moment for the request to succeed. This is why "our database is five-nines" is not the same claim as "our product is five-nines" — a product's effective availability is capped by the weakest, and multiplied down by every additional, component in its critical path. Teams that quote a headline availability number for a whole system are almost always quoting the target, not a number that automatically follows from the parts.

## Why redundancy pushes the number back up

Series dependencies make things worse; **parallel, redundant** paths make them better, for the opposite reason — a parallel path only fails when *every* option in it fails simultaneously, rather than succeeding only when *every* option works. If a request can be served by either of two independent, equally-capable replicas, each 99% available, the chance that *both* are down at the same moment is:

```
(1 − 0.99) × (1 − 0.99) = 0.01 × 0.01 = 0.0001 → 99.99% combined availability
```

Two merely-"two-nines" replicas, combined redundantly, behave like a single "four-nines" resource — as long as their failures are genuinely independent. That caveat matters: if both replicas sit in the same data center and a single power outage takes out both at once, their failures are correlated, not independent, and the math above overstates the real availability. This is the underlying reason multi-region and multi-availability-zone deployments exist — independence has to be engineered, not assumed.

## SLA vs SLO, briefly

Two related terms get used almost interchangeably in casual conversation but mean different things in practice. An **SLA** (service level agreement) is the external, often contractual promise made to a customer — "99.9% monthly uptime, or you get a service credit." An **SLO** (service level objective) is the internal target a team actually operates against day to day, and it's normally set *stricter* than the SLA on purpose, so that a bad week of internal performance shows up as "we missed our SLO, let's investigate" well before it ever becomes "we breached our customer-facing SLA." A team promising 99.9% externally might run an internal SLO of 99.95%, treating the gap between the two as a buffer rather than a target. A separate lesson later in this roadmap covers how SLOs connect to concrete SLIs and error budgets in depth — the point to take from here is just that the externally-visible number and the internally-engineered target are two different things, set deliberately apart.

## Putting the number to use

The practical use of all this isn't reciting downtime tables — it's using the arithmetic to find where a system's availability promise is actually at risk. Before promising a customer 99.99%, trace the full dependency chain behind that promise, multiply the component availabilities together, and see whether the chain as built can even reach the number being promised, or whether a redundant path needs to be added somewhere in series with the weakest link before that promise is honest.
