---
title: "Monitoring, Logging, and Alerting"
description: Metrics, logs, and traces as observability's pillars, why structured logging matters, and what makes an alert actionable instead of noise.
roadmap: system-design
node: monitoring-logging-and-alerting
group: observability-and-security-at-scale
video:
  provider: youtube
  id: "juP9VApKy_I"
  start: 0
  end: null
creator:
  name: dotconferences
  channel_url: https://www.youtube.com/@dotconferences
duration_seconds: 1102
level: intermediate
prerequisites: []
takeaways:
  - Metrics, logs, and traces are three different telemetry shapes, not three competing tools, and a production system needs all three to answer different questions.
  - A metric is cheap to store and great for "is something wrong right now," but it can't tell you which specific request failed or why.
  - A log is a detailed record of one event, and it earns its keep only when it's structured as queryable fields instead of a free-text sentence.
  - A trace follows one request across every service it touches, which is exactly the tool for "why was this one request slow" — the next lesson covers it in full.
  - A good alert fires on symptoms a human needs to act on, not on every internal fluctuation, because an alert nobody trusts gets ignored.
  - Observability is the property of being able to ask new questions of your telemetry after an incident, not just watching a fixed set of dashboards.
check_questions:
  - q: Why isn't a single metrics dashboard enough to debug most production incidents?
    a: A metric tells you that error rate or latency moved, but it's an aggregate number that can't identify which request, which user, or which code path caused the move, so you still need logs or traces to find the specific cause.
  - q: What's the practical difference between a log line and a structured log entry?
    a: A log line is a free-text sentence that a human can read but a machine can only grep, while a structured log entry stores the same event as named fields (timestamp, request ID, status code, latency) that a query engine can filter, aggregate, and join on.
  - q: What separates an alert worth waking someone up for from a noisy one?
    a: An actionable alert ties directly to user-visible symptoms (errors, latency, availability) and comes with a clear next step, while a noisy alert fires on internal fluctuations that don't require immediate human action and teaches the on-call engineer to ignore pages.
deeper:
  - label: "Applied Alerting Philosophy"
    url: "https://www.youtube.com/watch?v=JhxfZ0VIPP0"
  - label: "An introduction to monitoring and alerting with timeseries at scale, with Prometheus"
    url: "https://www.youtube.com/watch?v=gNmWzkGViAY"
related_formulas: []
---

## Three pillars, one job: knowing what your system is doing

For most of the history of running production software, "monitoring" meant a handful of dashboards and a paging system, and "logging" meant a folder of text files you `grep`ped through after something broke. Those two jobs have since split into three distinct kinds of telemetry — metrics, logs, and traces — that together get bundled under the word **observability**. They aren't competing approaches; they answer different questions, and a system that only has one of them is flying partially blind.

A **metric** is a number sampled over time: requests per second, p99 latency, queue depth, percentage of 5xx responses. Metrics are cheap to store because they're pre-aggregated — a counter doesn't remember every individual request, just the running total — which is exactly why they're the right tool for dashboards and for answering "is something wrong right now, and how wrong." What a metric can never tell you is *which* request was slow or *why*. If your p99 latency metric jumps from 80ms to 900ms, you know you have a problem, but the number itself is silent on the cause.

A **log** is a record of one discrete event: a request came in, a database call took 340ms, a payment failed with error code `INSUFFICIENT_FUNDS`. Logs carry the detail that metrics throw away. A **trace** follows a single request as it hops across every service involved in handling it, which is the tool built specifically for "why was this one request slow, and which hop caused it" — that's substantial enough to be its own lesson next, so this one only needs to place it on the map: traces are the third pillar, and they exist because neither a metric nor an isolated log line can show you the shape of a request's journey through a distributed system.

## Structured logging turns search into a query, not a guess

The habit that makes logs actually useful at scale is writing them as **structured data** instead of free-text sentences. Compare:

```
2026-03-14 09:12:03 ERROR Payment failed for user 88213 order 55012: insufficient funds
```

against a structured equivalent:

```json
{"ts":"2026-03-14T09:12:03Z","level":"error","event":"payment_failed","user_id":88213,"order_id":55012,"reason":"insufficient_funds","request_id":"a1b2c3"}
```

The free-text version is readable by a human scrolling a terminal, but it's nearly useless at scale: finding "every payment failure for this user in the last hour, broken down by reason" means writing a fragile regex and hoping the message format never changes. The structured version is queryable: `event:payment_failed AND user_id:88213`, aggregated, filtered, joined with other fields, without anyone having to parse a sentence. The `request_id` field is what lets you pull every log line that belongs to one request across every service it touched — the same correlation idea a trace ID implements more formally. Structured logging doesn't require exotic tooling; it just requires deciding, from the first service you write, that log lines are data with a schema, not prose for a human to read later.

## What makes an alert worth paging someone for

Collecting telemetry is only half the job — the other half is deciding when a human needs to be interrupted. This is where most observability setups quietly fail: a team wires up alerts on anything they can measure, and within a few months the on-call engineer is getting paged for CPU blips that resolved themselves ninety seconds later. The result is **alert fatigue** — pages get silenced, Slack channels get muted, and eventually a real incident gets missed in the noise.

A useful discipline is to alert on **symptoms**, not **causes**. "Checkout error rate exceeded 2% for five minutes" is a symptom — it's user-visible, it's unambiguous, and it demands action. "CPU usage on `worker-17` exceeded 80%" is usually a cause, and causes fluctuate constantly without ever becoming a user-visible problem; alerting on them pages someone for information, not for an incident. The next lesson on error budgets takes this further by giving "how much is too much" an actual number, but even without that machinery, the test for any alert should be: if this fires at 3am, is there something a human must do right now, and do they know what it is? An alert without a clear next step is a notification, not an alert, and belongs on a dashboard instead of in a pager.

Put together, metrics tell you something is wrong, logs and traces tell you what and why, and a disciplined alerting policy is what decides which of those "somethings" is worth a human's immediate attention. Get the first two right and skip the third, and you'll have a beautiful dashboard nobody looks at until it's too late; get the third right without the first two, and you'll have alerts with nothing underneath them to diagnose the problem they just reported.
