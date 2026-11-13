---
title: "Distributed Tracing"
description: Why one request spanning a dozen services needs a trace ID and per-hop spans, and how a trace waterfall reveals which hop is slow.
roadmap: system-design
node: distributed-tracing
group: observability-and-security-at-scale
video:
  provider: youtube
  id: "WS-eL_mkfWU"
  start: 0
  end: null
creator:
  name: CodeSnippet by Chetan
  channel_url: https://www.youtube.com/@CodeSnippetByChetanGhate
duration_seconds: 1012
level: intermediate
prerequisites:
  - monitoring-logging-and-alerting
takeaways:
  - A single user-facing request in a microservices system can fan out into dozens of internal calls, and no individual service's logs show the whole journey.
  - A trace ID is generated once, at the edge, and propagated through every downstream call so every service involved can tag its own records with it.
  - A span represents one unit of work done by one service — a database query, an RPC call, a cache lookup — with its own start time and duration.
  - Spans nest into a tree — a parent span (say, the API gateway's handling of the request) contains child spans for every downstream call it made.
  - A trace waterfall lays every span out on a shared timeline, so the service that's actually eating the latency is visually obvious instead of a guess.
  - Propagating trace context across process and network boundaries is itself engineering work — every hop has to read the incoming trace ID and pass it along, including through queues and async jobs.
check_questions:
  - q: Why can't you debug a slow checkout request just by reading the checkout service's own logs?
    a: The checkout service's logs only show its own slice of the work; if the actual delay happened three hops downstream in an inventory or payment service, that service's logs are the ones that would show it, and nothing in the checkout service's logs points you there without a shared trace ID connecting them.
  - q: What's the relationship between a trace and a span?
    a: A trace is the full record of one request's journey through a system, and it's made up of spans, where each span is one unit of work performed by one service, timestamped with its own start and duration and usually linked to a parent span.
  - q: What does a trace waterfall view show that a list of log lines doesn't?
    a: It lays every span on a shared timeline so their relative start times and durations are visible at a glance, turning a vague sense that something downstream was slow into a precise fact — this exact service call, starting at this exact offset, took 400 of the request's 420 total milliseconds.
deeper:
  - label: "Observability at Uber Engineering: Past, Present, and Future"
    url: "https://www.youtube.com/watch?v=2JAnmzVwgP8"
related_formulas: []
---

## One request, a dozen services, and nobody with the full picture

A monolith makes debugging latency simple in one specific way: there's only one process, so one set of logs and one profiler can show you the entire life of a request. Break that monolith into a dozen microservices — an API gateway, an auth service, a product catalog, an inventory check, a pricing engine, a payments service — and that simplicity disappears. A single checkout request might now fan out into fifteen or twenty internal calls across as many services, each running on its own hosts, each writing its own logs, each with no inherent knowledge of the other fourteen.

When that checkout request takes 2 seconds instead of the usual 200 milliseconds, where do you look? The checkout service's own logs show it made calls to inventory and payments and waited on both, but not which one was slow, or whether the slowness was in that service itself or something it in turn called. Scaling this problem out: in a system with enough services, a meaningful fraction of your "slow request" investigations would otherwise mean manually correlating log timestamps across a dozen unrelated systems, guessing at which ones even participated in a given request. Distributed tracing exists to remove the guessing.

## Trace IDs and spans: stitching one request back together

The fix is to generate a single **trace ID** the moment a request enters the system — typically at the load balancer or API gateway — and propagate it through every subsequent call that request triggers. When the gateway calls the auth service, it attaches the trace ID to that call (commonly as an HTTP header, or a field in an RPC's metadata). When auth calls a token-validation service, it forwards the same trace ID again. By the time the request has fanned out through fifteen hops, all fifteen hops are tagging their own records with the identical ID, even though none of those services otherwise know about each other.

Each individual hop's unit of work is recorded as a **span**: "the inventory service checked stock for SKU 4471, starting at T+140ms, taking 60ms." A span records at minimum a name, a start time, and a duration, and it usually carries a reference to its **parent span** — the span that caused it to happen. The checkout service's top-level span is the parent of the inventory-check span and the payment-authorization span, because the checkout service is what called both. Spans nest the same way function calls nest in a single-process stack trace; the difference is that each "frame" here might be running on a different machine, written in a different language, miles apart.

Collect every span that shares one trace ID and you've reconstructed the request's complete path through the system — which services it touched, in what order, and for how long each one ran — purely from data each service emitted independently, with no service needing to know the others existed.

## Reading a trace waterfall

The standard way to visualize a trace is a **waterfall**: a horizontal bar per span, all placed on one shared timeline, with child spans indented under their parents. A healthy request might show a gateway span spanning the whole width, with five short child spans packed near the start and end, each taking a few milliseconds. A slow request shows the same shape with one bar stretched far wider than the rest — and that bar's label tells you exactly which service and which operation is the bottleneck, with no correlation guesswork required.

This is where tracing earns its keep over logs and metrics alone. A latency metric tells you the p99 moved. A trace waterfall for one of those slow requests tells you it moved because the pricing service's call to a third-party tax API took 1.8 of the request's 2 seconds — a fact that would otherwise require cross-referencing timestamps across two services' independent log streams and hoping the clocks agree.

## The part that's actually hard: propagation

The conceptually simple part of tracing — tagging a span with a trace ID — is not the hard part in practice. The hard part is making sure that ID survives every kind of boundary a request can cross: synchronous HTTP calls, gRPC calls, messages dropped onto a queue and picked up by a worker minutes later, retries, fan-out to multiple services in parallel. Each of these needs explicit code (often supplied by an instrumentation library, commonly built on the OpenTelemetry standard today) that reads the incoming trace context and re-attaches it to whatever the service does next. Miss one hop — a queue consumer that doesn't forward the trace header, say — and the trace simply stops at that point, showing a gap instead of the work that happened beyond it. Tracing's practical difficulty is less "how do we represent a trace" and more "how do we guarantee every service, written by every team, propagates it correctly."
