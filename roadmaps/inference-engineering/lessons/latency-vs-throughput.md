---
title: Latency versus throughput
description: Batching more requests raises aggregate throughput but can raise individual latency — tuning a serving system means picking a point on that curve.
roadmap: inference-engineering
node: latency-vs-throughput
group: latency-and-throughput
video:
  provider: youtube
  id: "DeFF3J8T5Pk"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 967
level: advanced
prerequisites: [tokens-per-second-and-throughput]
takeaways:
  - The right way to compare serving systems is to pick a latency target first, then ask how much throughput each system can sustain while staying within it — not to compare raw throughput numbers alone.
  - Past a certain point, adding more concurrent requests to a system causes latency to rise sharply rather than gradually, because requests start queueing faster than the system can clear them.
  - Tuning a serving system is choosing a deliberate point on the latency-throughput curve for your product's needs, not maximising either number in isolation.
check_questions:
  - q: Why is comparing two systems' raw throughput numbers alone a poor way to choose between them?
    a: A system can post a higher throughput number by tolerating much worse latency, which may be unacceptable for your product. The fair comparison fixes a latency target first and then asks which system sustains more throughput within that target.
  - q: Why does latency rise sharply rather than gradually once a system is pushed past its capacity?
    a: Once incoming requests arrive faster than the system can clear them, a queue begins to grow, and each newly arriving request has to wait behind an ever-lengthening line in front of it. This queueing effect compounds, which is why latency explodes rather than degrading smoothly.
deeper: []
related_formulas: [littles-law]
---

## Comparing systems the right way

It is tempting to compare two serving systems, or two configurations of the
same system, by their raw throughput numbers — whichever produces more
tokens per second wins. This is a poor comparison on its own, because
throughput can always be bought by tolerating worse latency: push enough
concurrent requests through a system and its aggregate throughput climbs,
while any individual request waits longer and longer for its turn. The fair
comparison fixes a latency target first — say, a p95 time to first token
under two seconds — and then asks how much throughput each system or
configuration can sustain while staying inside that target.

## Why latency explodes rather than degrading gracefully

A serving system has a finite capacity to process requests. Below that
capacity, adding a few more concurrent requests raises latency only
slightly, since there is spare capacity to absorb them. Once incoming
demand exceeds what the system can actually process, a queue starts to form,
and it keeps growing for as long as demand exceeds capacity — each new
request waits behind an ever-lengthening line of requests ahead of it. This
is why the latency curve, plotted against increasing load, stays nearly flat
for a while and then rises almost vertically: the system has crossed from
"keeping up" to "falling behind," and those two regimes look completely
different.

[Little's Law](/formulas/littles-law) describes this precisely: the average
number of requests in the system equals the arrival rate multiplied by the
average time each one spends there. Once arrivals outpace the rate at which
requests complete, that average time spent in the system — which includes
however long a request waits in queue — grows without bound.

## Reading a latency-throughput curve in practice

A real benchmark of this tradeoff typically produces a curve rather than a
single number: throughput plotted on one axis, the latency required to
achieve it on the other. Near the left side of that curve, small increases in
throughput cost very little extra latency — there is spare capacity being
used efficiently. Further along, each additional unit of throughput starts
costing noticeably more latency, and past a certain point the curve bends
sharply upward, which is the queueing effect described above taking hold.
Reading this curve for a specific system tells you directly where the
genuinely efficient operating region ends — and operating anywhere past that
bend means accepting a disproportionate latency cost for comparatively little
additional throughput, which is rarely the right trade once you can see the
curve laid out this way.

## Tuning is choosing a point, not maximising a number

A production serving system is almost never configured for maximum possible
throughput, because that configuration accepts whatever latency results,
however bad. It is configured for the highest throughput achievable while
staying inside a latency target the product actually needs — a real,
deliberate choice specific to what is being built, not a universal setting
to copy from somewhere else. A latency-sensitive interactive product and a
throughput-sensitive batch pipeline will reasonably land on very different
points along this same curve for the exact same hardware and the exact same
model.
