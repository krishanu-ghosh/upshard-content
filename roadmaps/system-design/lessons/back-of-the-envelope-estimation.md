---
title: "Back-of-the-Envelope Estimation"
description: Fast, rough math for QPS, storage, and bandwidth using powers-of-two/ten shortcuts — a habit for catching a wrong architecture early.
roadmap: system-design
node: back-of-the-envelope-estimation
group: design-fundamentals-and-estimation
video:
  provider: youtube
  id: "0myM0k1mjZw"
  start: 0
  end: null
creator:
  name: Gaurav Sen
  channel_url: https://www.youtube.com/@gkcs
duration_seconds: 792
level: intermediate
prerequisites:
  - latency-vs-throughput
  - what-is-system-design
takeaways:
  - Back-of-the-envelope estimation converts a vague spec ("a few million users") into rough numbers for QPS, storage, bandwidth, and memory, using powers-of-two/ten shortcuts instead of a calculator.
  - The goal is the right order of magnitude, not precision — knowing whether you need gigabytes or petabytes changes the architecture; knowing the exact byte count never does.
  - Average QPS comes from daily active users times actions per user, divided by 86,400 seconds in a day; peak QPS is usually 2-5x the average to account for uneven traffic.
  - Storage estimates multiply records (or events) per day by bytes per record, then by a retention period, to get a total that tells you whether a single database will do or you need sharding from day one.
  - Memorized reference points — a few KB for a tweet, roughly 1ms for an SSD read, roughly 150ms for a cross-continent round trip — let you sanity-check a number in your head without looking anything up.
  - Catching that a design implies 50 petabytes of storage or 2 million QPS before writing a line of code is the entire point — the estimate exists to reject a wrong architecture early, not to produce a number anyone will hold you to later.
check_questions:
  - q: What is the actual goal of a back-of-the-envelope estimate — a precise number, or something else?
    a: The goal is the correct order of magnitude, not a precise figure — the estimate needs to tell you whether a system is a megabyte-scale, gigabyte-scale, or petabyte-scale problem, because that distinction changes the architecture, while exact precision beyond that doesn't change any decision.
  - q: How would you roughly estimate average queries per second for a service with 10 million daily active users who each perform 5 actions a day?
    a: Multiply 10 million users by 5 actions to get 50 million actions per day, then divide by the roughly 86,400 seconds in a day, giving an average of a little under 600 queries per second.
  - q: Why is peak QPS usually estimated as several times the average QPS rather than equal to it?
    a: Because real traffic isn't evenly spread across the day — it clusters around certain hours or events — so a system sized only for the daily average would be overwhelmed during the busiest periods, and a multiplier of roughly 2-5x the average is a common rough adjustment for that unevenness.
deeper: []
related_formulas: []
---

## Why estimate before you design

A system design conversation usually starts with a vague spec: "a URL shortener," "a chat app for a few million users." Before any component gets chosen — one database or a sharded cluster, a single server or a fleet behind a load balancer — it helps enormously to turn that vague spec into a few rough numbers: how many requests per second, how much data accumulates, how much bandwidth gets used. This is **back-of-the-envelope estimation**: quick math, done with round numbers and memorized shortcuts, that's accurate enough to tell you which architecture family you're in, even though it would be useless as an actual capacity-planning document.

The payoff is catching a wrong turn early. If a rough estimate says a system needs to handle 2 million writes per second, that number alone rules out "a single PostgreSQL instance" before anyone spends an hour designing around it. If it says the system needs 500MB of storage total, that same number rules out building a sharding strategy nobody needs yet. Estimation is a filter on architecture choices, not a deliverable.

## The shortcuts that make this fast

A few memorized reference numbers make this math quick enough to do in your head or on a whiteboard:

**Powers of two, approximated as powers of ten.** 2^10 (1,024) is close enough to 1,000 to call a kilo-something; 2^20 to a million (mega); 2^30 to a billion (giga); 2^40 to a trillion (tera). You don't need the exact byte counts — treating 2^10 as "about a thousand" throughout a calculation keeps the arithmetic simple without meaningfully changing the order of magnitude.

**Seconds in a day ≈ 86,400 ≈ 10^5.** This single number is the hinge for converting almost any "per day" figure into a "per second" figure, which is usually the number that actually determines server count.

**Latency numbers worth having memorized**, roughly:

| Operation | Rough latency |
|---|---|
| L1/L2 cache reference | ~1 nanosecond |
| Main memory (RAM) reference | ~100 nanoseconds |
| SSD random read | ~0.1-1 millisecond |
| Round trip within the same datacenter | ~0.5 millisecond |
| Disk seek (spinning disk) | ~10 milliseconds |
| Round trip across continents (e.g. US to Europe) | ~100-150 milliseconds |

The exact figures drift with hardware generations, but the *relative* gaps don't: memory is roughly 100x faster than SSD, which is roughly 10-100x faster than a cross-datacenter network call. Knowing that a design does five sequential cross-continent round trips before responding (5 x ~150ms = 750ms) tells you it has a latency problem before you've measured anything.

## Working a QPS and storage estimate

Take a concrete example: a photo-sharing service with 50 million daily active users (DAU), where each user uploads 2 photos a day on average, and each photo plus its metadata is about 2MB.

**Average QPS (writes):**
```
50,000,000 users x 2 photos/day = 100,000,000 uploads/day
100,000,000 / 86,400 seconds/day ≈ 1,160 uploads/second (average)
```

**Peak QPS:** Traffic isn't flat across 24 hours — it clusters during waking hours in each timezone. A common rough adjustment is to assume peak load is 2-5x the average; using 3x here:
```
1,160 x 3 ≈ 3,500 uploads/second (peak)
```
That number — a few thousand QPS at peak — already tells you this needs more than one application server and probably a load balancer in front of several, but it's nowhere near the scale (hundreds of thousands of QPS) that would force a conversation about sharding the write path from day one.

**Storage for one year:**
```
100,000,000 uploads/day x 2MB/upload = 200,000,000 MB/day = 200TB/day
200TB/day x 365 days ≈ 73,000TB ≈ 73PB/year
```
That's a very different story — tens of petabytes a year is firmly in "this needs object storage designed for that scale, with a CDN and a deliberate storage tiering strategy," not "a single database's disk." The QPS number said "a modest fleet of servers is fine"; the storage number said "the storage layer needs real architecture." Two numbers from the same spec, pointing at two different parts of the design, is exactly the kind of signal this exercise exists to produce.

## Precision is not the point

Notice what's missing from the math above: no attempt to account for photo compression ratios varying by device, no correction for weekends vs. weekdays, no discussion of exactly how many bytes of metadata accompany each image. Back-of-the-envelope estimation deliberately ignores precision that wouldn't change a decision. The question being answered is always "which order of magnitude, and does that change what I'm about to design," not "what is the exact number." If a more precise figure would change the chosen architecture, it's worth getting — if it wouldn't, the imprecision is free. That discipline — doing just enough math to validate or reject a design, and no more — is the actual skill, and it's one of the few system-design habits that pays off identically whether you're in an interview or sizing out a real system before it's built.
