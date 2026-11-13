---
title: "Circuit Breakers and Retries"
description: The circuit breaker state machine, retry with exponential backoff and jitter, and why naive retries turn a blip into a full outage.
roadmap: system-design
node: circuit-breakers-and-retries
group: reliability-and-resilience-patterns
video:
  provider: youtube
  id: "CZ3wIuvmHeM"
  start: 0
  end: null
creator:
  name: InfoQ
  channel_url: https://www.youtube.com/@infoq
duration_seconds: 3193
level: intermediate
prerequisites:
  - availability-patterns-and-failover
takeaways:
  - A circuit breaker wraps a risky call and tracks its recent failure rate, so a struggling dependency gets cut off instead of called into the ground.
  - The breaker has three states — closed (calls flow normally), open (calls fail instantly without touching the dependency), and half-open (a trickle of calls tests recovery) — and it moves between them automatically based on failure counts and timers.
  - Retries turn a single transient failure into a success, but only when the operation is idempotent and the backoff grows between attempts instead of firing immediately.
  - Jitter — adding randomness to each retry's delay — exists specifically to stop every client from retrying at the exact same moment and recreating the spike that caused the failure.
  - A retry storm happens when a struggling service gets hit by everyone's retries on top of its already-degraded capacity, which is precisely the failure mode a circuit breaker is designed to prevent.
  - Retries and circuit breakers are complementary, not interchangeable, since retries handle a call that might succeed on a second try, while breakers handle a dependency that is down and should not be called at all.
check_questions:
  - q: Why does a circuit breaker have a half-open state instead of just flipping straight from open back to closed?
    a: Flipping straight back to closed would send the dependency its full traffic the instant the timeout expires, which could overwhelm a service that's only partially recovered; half-open lets a small number of trial requests confirm recovery before traffic ramps back up.
  - q: A client retries a failed request three times, each exactly 1 second apart, with no jitter. What goes wrong when 10,000 clients hit the same failure at once?
    a: All 10,000 clients retry at the same three moments, so the downstream service sees three synchronized traffic spikes instead of steady load, which can keep it failing indefinitely even after the original cause clears up.
  - q: Why is it unsafe to automatically retry a non-idempotent operation like "charge this credit card"?
    a: If the first attempt actually succeeded but the client never saw the response (e.g., the network dropped the reply), a retry would charge the card a second time, so retries are only safe for operations that can be repeated without changing the outcome.
---

## The problem retries and breakers both solve

Any call across a network can fail for reasons that have nothing to do with correctness: a packet gets dropped, a downstream service is garbage-collecting, a load balancer is mid-deploy. These are *transient* faults — try again a moment later and they're often gone. But "try again" is a trap if you do it carelessly, because the thing you're retrying might not be transient at all; it might be a dependency that's genuinely overloaded, in which case more retries are exactly the wrong medicine. Circuit breakers and retries are the two tools that handle these two different situations, and a resilient system needs both, used correctly.

## The circuit breaker state machine

A circuit breaker sits in front of a risky call — usually a network call to another service — and tracks how often it's been failing. It has three states:

- **Closed** — the normal state. Calls pass through to the real dependency, and the breaker counts failures in a rolling window.
- **Open** — once failures cross a threshold (say, 50% of calls over the last 10 seconds, with a minimum volume so one unlucky request doesn't trip it), the breaker stops forwarding calls entirely. It returns an error — or a fallback — immediately, without touching the dependency. This is the whole point: a service that's drowning doesn't need more load added to it while it recovers.
- **Half-open** — after a cooldown period, the breaker lets a small number of requests through as a probe. If they succeed, it closes again and resumes normal traffic. If they fail, it reopens and restarts the timer.

This cycle — closed → open → half-open → (closed or open) — is the entire mechanism. The two parameters that matter in practice are the failure threshold (how bad does it have to get before tripping) and the cooldown (how long to wait before probing). Set the threshold too sensitive and the breaker trips on routine noise; set the cooldown too short and half-open probes become their own mini retry storm.

What a circuit breaker buys you is *latency* as much as availability. Without one, a call to a dead dependency still has to time out — often 10-30 seconds — before your code finds out it failed, and every thread or connection tied up waiting is one that can't serve another request. An open breaker fails in microseconds. That's the difference between a slow page and a thread-pool exhaustion cascade that takes down the service that was calling the dead one.

## Retry with exponential backoff and jitter

A retry is simpler in concept: the call failed, so do it again. The two things that make a retry policy safe rather than dangerous are backoff and jitter.

**Exponential backoff** grows the wait between attempts — 1s, then 2s, then 4s, then 8s, typically capped at some maximum — instead of hammering the dependency at a fixed interval. The logic is that if a service is overloaded, retrying immediately adds to the overload; waiting longer each time gives it room to drain its queue.

**Jitter** solves a problem backoff alone doesn't: synchronization. If every client that failed at the same moment backs off on the exact same schedule, they all retry at the exact same moment too — you've just turned one spike into three or four, perfectly synchronized spikes. Full jitter fixes this by picking the actual delay randomly between 0 and the current backoff ceiling: `delay = random(0, min(cap, base * 2^attempt))`. Now failed clients fan out across the window instead of retrying in lockstep.

A retry policy also needs a ceiling on attempts and, ideally, a total deadline for the overall operation — "retry up to 3 times, but give up regardless if 5 seconds have passed" — so that retries don't silently turn a fast failure into a slow one from the caller's point of view.

## Retry storms: when the cure becomes the disease

The failure mode both of these patterns exist to prevent is the **retry storm**: a dependency degrades slightly, calls start timing out, every caller retries, the extra retry traffic pushes the dependency from "slightly degraded" to "fully down," which causes more timeouts, which causes more retries. This is a positive feedback loop, and it's one of the most common ways a minor blip becomes a major outage — the dependency might have recovered on its own in seconds if retry traffic hadn't kept it pinned down.

This is exactly why retries and circuit breakers need to work together rather than independently: a retry policy without a circuit breaker will cheerfully keep retrying into a dependency that's completely down, at full volume, for as long as the deadline allows. Wrapping the retried call in a circuit breaker means that once the failure rate crosses the threshold, retries stop generating new load — the breaker is open, calls fail fast, and the dependency gets the silence it needs to recover. The two patterns aren't redundant: retries smooth over single blips, and the breaker is the circuit-level safeguard that keeps a sustained failure from turning every caller's retry logic into a denial-of-service attack on your own infrastructure.
