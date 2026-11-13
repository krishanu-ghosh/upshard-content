---
title: "Rate Limiting Algorithms"
description: Token bucket, leaky bucket, fixed-window, and sliding-window rate limiters — how each one actually behaves at a burst boundary.
roadmap: system-design
node: rate-limiting-algorithms
group: reliability-and-resilience-patterns
video:
  provider: youtube
  id: "eR66m7TaV5A"
  start: 0
  end: null
creator:
  name: Piyush Garg
  channel_url: https://www.youtube.com/@piyushgargdev
duration_seconds: 1173
level: intermediate
prerequisites:
  - backpressure-and-flow-control
takeaways:
  - A rate limiter's job is to cap how many requests a client can make in a window, and every algorithm is really just a different way of counting that's traded off against memory cost and burst behavior.
  - Token bucket lets a client spend saved-up tokens in a burst, then throttles it back to the average refill rate once the bucket is empty.
  - Leaky bucket forces a constant output rate regardless of how bursty the input is, which smooths traffic but adds queueing delay that token bucket doesn't.
  - A fixed window counter is cheap but lets exactly double the allowed rate slip through at the boundary between two windows.
  - The sliding window log is perfectly accurate but stores a timestamp per request; the sliding window counter approximates it cheaply by blending the current and previous fixed windows.
  - Choosing an algorithm is a tradeoff between memory per client, tolerance for short bursts, and how precisely the limit has to be enforced at the edges of a time window.
check_questions:
  - q: Why can a fixed-window counter let through twice its stated limit in a short burst?
    a: If the limit is 100 requests per minute, a client can send 100 requests in the last second of one window and another 100 in the first second of the next window, so 200 requests land within roughly two seconds even though each window individually stayed under its cap.
  - q: What's the key behavioral difference between token bucket and leaky bucket when a client suddenly sends 50 requests at once after being idle?
    a: Token bucket lets the burst through immediately up to however many tokens have accumulated, while leaky bucket queues the requests and releases them at its fixed drain rate, smoothing the burst into a steady trickle instead of passing it straight through.
  - q: Why does the sliding window log algorithm need more memory per client than the sliding window counter?
    a: The log stores an individual timestamp for every request in the current window so it can count exactly how many fall inside any rolling interval, while the counter only keeps two aggregate counts (current and previous window) and estimates the rest, trading some precision for a fixed, small memory footprint.
---

## What every rate limiter is actually counting

Strip away the implementation details and a rate limiter answers one question on every incoming request: "has this client already used up its allowance for the relevant window of time?" The interesting differences between algorithms come from how they define "window" and how precisely they count inside it. Some trade precision for cheap, fixed-size storage; others trade storage for exactness at the burst boundary. Understanding the four common algorithms means understanding exactly where each one draws that line.

## Token bucket

Picture a bucket that holds up to *N* tokens, refilling at a steady rate (say, 10 tokens per second), and every request costs one token. If the bucket has tokens, the request proceeds and a token is removed; if it's empty, the request is rejected or delayed.

The defining behavior: a client that's been idle accumulates tokens up to the bucket's capacity, and can then spend all of them in a single burst. If the bucket holds 100 tokens and refills at 10/second, a client idle for 10+ seconds can fire 100 requests in the same instant and all of them succeed — then it's throttled back down to the steady 10/second refill rate until it either slows down or waits again. Token bucket is the right choice when you want to permit short, legitimate bursts (a user double-clicking, a batch job catching up) while still bounding the long-run average rate.

## Leaky bucket

Leaky bucket flips the mental model: instead of a reservoir of permission, it's a queue with a hole in the bottom that drains at a fixed rate. Requests arrive and get queued; the queue drains (i.e., requests get processed) at a constant rate no matter how bursty the arrivals were. If the queue fills up, new requests are dropped.

The defining behavior: output is *always* smooth, even when input isn't. Where token bucket says "a burst is fine, just not forever," leaky bucket says "there is no burst, ever — everything gets smoothed to the drain rate," at the cost of added latency for requests sitting in the queue. This makes leaky bucket a better fit for protecting something that genuinely can't handle bursts at all — a downstream system with a fixed, hard processing capacity — while token bucket suits interactive APIs where an occasional fast burst is a feature, not a risk.

## Fixed window counter

The simplest approach: divide time into fixed windows (e.g., each calendar minute), keep a counter per client per window, and reject once the counter hits the limit. It costs one integer per client and is trivial to implement.

Its flaw is the boundary effect. The counter resets exactly at the window edge with no memory of what happened in the previous window, so a client can spend its full allowance in the last moment of one window and its full allowance again in the first moment of the next. A "100 requests per minute" limit can, in the worst case, let 200 requests through within about two seconds straddling the boundary — double the intended rate, briefly but predictably, every single window.

## Sliding window log and sliding window counter

The **sliding window log** fixes the boundary problem with brute force: store the timestamp of every request a client has made, and on each new request, count how many stored timestamps fall within the last window-length of time (e.g., the last 60 seconds, measured continuously, not aligned to clock minutes). This is exactly accurate — there's no boundary to exploit because the window itself is always relative to "now" — but memory grows with request volume, since every timestamp has to be kept until it ages out.

The **sliding window counter** approximates the log cheaply: keep only two numbers per client, the count in the current fixed window and the count in the previous one, and estimate the sliding count as a weighted blend — for example, if you're 30% of the way into the current window, assume 70% of the previous window's requests still "count" against you: `estimated = previous_count * (1 - elapsed_fraction) + current_count`. This isn't exact (it assumes requests were spread evenly across the previous window, which they weren't), but it gets very close to the log's accuracy while costing a fixed two integers per client regardless of how many requests arrive — which is why it's the algorithm most production rate limiters actually ship.

## Picking one

In practice: fixed window if you can tolerate occasional double-rate bursts and want the cheapest possible implementation; token bucket if bursts are a desired feature of the API; leaky bucket if the protected resource needs genuinely constant-rate input; sliding window counter if you need near-exact enforcement without the unbounded memory of a full log. None of these is universally "correct" — the right pick depends entirely on whether a short burst at the limit's edge is a problem you need to solve or a behavior you're happy to allow.
