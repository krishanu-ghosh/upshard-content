---
title: "Health Checks and Chaos Engineering"
description: Liveness vs readiness checks that feed load balancers and orchestrators, and chaos engineering as deliberately testing whether resilience assumptions hold.
roadmap: system-design
node: health-checks-and-chaos-engineering
group: reliability-and-resilience-patterns
video:
  provider: youtube
  id: "6OIOpx_dVFY"
  start: 0
  end: null
creator:
  name: Midwest.io
  channel_url: https://www.youtube.com/@midwestio
duration_seconds: 2564
level: intermediate
prerequisites:
  - load-balancing-algorithms
  - bulkheads-and-graceful-degradation
takeaways:
  - A liveness check answers "is this process alive and not deadlocked," and failing it gets an instance restarted.
  - A readiness check answers "can this instance currently handle traffic," and failing it gets an instance pulled from load-balancer rotation without being restarted.
  - Conflating the two checks is a common bug — a liveness check that also tests a downstream database causes an orchestrator to kill and restart instances for a problem that restarting can't fix.
  - Health checks only verify what they test; a shallow check that just returns 200 OK can hide an application that's technically running but functionally broken.
  - Chaos engineering means deliberately injecting real failures into a system to verify that the resilience mechanisms you built — retries, breakers, bulkheads, failover — actually work, rather than assuming they do.
  - Chaos experiments start small and targeted (one instance, a fraction of traffic) with an easy abort path, not as a surprise blast against full production.
check_questions:
  - q: Why does Kubernetes use two separate probes — liveness and readiness — instead of one?
    a: Because the right response to each kind of failure is different — a deadlocked process should be killed and restarted (liveness), while an instance that's just temporarily unable to serve traffic, for example still warming its cache, should be left running but taken out of rotation until it's ready (readiness), and a single probe can't distinguish those cases.
  - q: A liveness check queries the application's primary database to confirm connectivity. The database goes down. What goes wrong?
    a: Every instance's liveness check now fails even though the instances themselves are perfectly healthy, so the orchestrator restarts all of them — which does nothing to fix the database and just adds restart churn and cold-start latency on top of an outage that was already happening.
  - q: What's the actual goal of a chaos engineering experiment — is it to break production?
    a: No; the goal is to verify, under controlled and reversible conditions, that a specific resilience assumption (like "if this dependency fails, the circuit breaker trips and the fallback kicks in") actually holds in the real system, so that the first time that assumption is tested isn't during an uncontrolled real outage.
---

## Two different questions, two different checks

Load balancers and orchestrators (Kubernetes, ECS, a classic load balancer's health-check config) all need an automated answer to "should I send traffic to this instance right now?" — but that single-sounding question actually hides two distinct ones, and conflating them is one of the most common operational mistakes in running a service.

**Liveness** asks: is this process fundamentally alive, or has it deadlocked / crashed / hung in a way it will never recover from on its own? The only correct response to a failed liveness check is to kill the instance and start a fresh one — restarting is the fix, because the process is presumed unrecoverable.

**Readiness** asks: right now, in this moment, can this instance actually handle a request well? An instance can be perfectly alive and still answer "no" here — it might be warming a cache, finishing startup, waiting on a downstream dependency it needs before serving traffic, or deliberately pulling itself out of rotation while it drains in-flight requests before a shutdown. The correct response to a failed readiness check is *not* to restart anything — it's to stop routing new traffic to that instance until it reports ready again.

The practical failure mode when these get mixed up: someone wires the liveness check to also verify a downstream dependency — "liveness check queries the database to make sure everything's connected." The moment that database has a blip, every single instance's liveness check fails simultaneously, and the orchestrator responds by killing and restarting the entire fleet. This doesn't fix the database — it can't — and it adds a fleet-wide restart storm, with all the cold-start latency and cache-warming that implies, directly on top of an outage that was already in progress. The dependency check belonged in readiness, where the correct response (stop routing traffic here, but leave the process running so it can recover instantly once the dependency returns) actually matches the failure.

## What a health check actually verifies — and what it doesn't

A health check is only as good as what it tests. The cheapest possible implementation — an endpoint that always returns `200 OK` as long as the HTTP server is up — tells you the process can accept a TCP connection and nothing more. It will happily report healthy while the application is deadlocked on every actual request, its thread pool is exhausted, or a critical internal queue is backed up and growing unbounded. A meaningful readiness check needs to exercise enough of the real request path — a lightweight query against the actual database connection pool, a check that the queue isn't past some threshold — to catch the failures that matter, while staying cheap enough to run every few seconds without becoming a load problem of its own.

## Chaos engineering: testing the assumptions, not hoping they hold

Every pattern covered elsewhere in this group — circuit breakers, retries, bulkheads, fallbacks — is built on an assumption: "if X fails, the system will respond by doing Y." Those assumptions are usually based on code review and maybe a unit test, which means the first time most of them get tested against a *real* failure is during an actual incident. Chaos engineering exists to move that test earlier, under conditions the team controls.

The core idea is to deliberately and safely inject the kind of failure the system is supposed to handle gracefully, and then observe whether it actually does. Does killing one instance of a service actually cause its load balancer to route around it within the expected time, or does it take 30 seconds of 500s first? Does injecting latency into a dependency actually trip the circuit breaker at the threshold it was configured for, or does the breaker never trip because the failure doesn't look the way its detection logic expects? These are empirical questions, and the only reliable way to answer them is to actually cause the failure and watch what happens — not to read the code and assume it's correct.

A responsible chaos experiment has a few consistent properties: it starts with a specific, falsifiable hypothesis ("if we kill 10% of instances, p99 latency stays under 500ms"), it's scoped as small as possible to still be a valid test (one instance or a small percentage of traffic, not the whole fleet), and it has a fast, reliable way to abort and revert if the blast radius turns out to be bigger than expected. The point is never "see what breaks" for its own sake — it's "confirm this specific safety mechanism works, under real conditions, while someone is watching and able to stop it," which is a fundamentally different and much safer activity than it sounds like from the name.

## Why these two topics belong together

Health checks and chaos engineering are really the same instinct applied at different timescales. A readiness check is a continuous, automated micro-experiment running every few seconds, asking "is this instance okay right now" and acting on the answer immediately. A chaos experiment is a deliberate, occasional macro-experiment asking the bigger question — "does this whole system's failure response actually work" — under conditions designed to make the answer trustworthy rather than assumed.
