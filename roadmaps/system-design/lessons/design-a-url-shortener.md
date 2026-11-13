---
title: "Case Study: Design a URL Shortener"
description: A worked design for a bit.ly-style shortener, covering short-code generation, collision handling, and the cache that keeps redirects fast.
roadmap: system-design
node: design-a-url-shortener
group: case-studies
video:
  provider: youtube
  id: "AVztRY77xxA"
  start: 0
  end: null
creator:
  name: codeKarle
  channel_url: https://www.youtube.com/@codeKarle
duration_seconds: 1476
level: advanced
prerequisites:
  - sql-vs-nosql-and-data-modeling
  - distributed-caching-systems
takeaways:
  - A URL shortener is a read-heavy, latency-sensitive lookup service in disguise, so almost every design decision favors read speed over write elegance.
  - Base62 encoding of a globally unique counter produces short, collision-free codes without the retry loop that random-hash generation needs.
  - A distributed counter (ranges handed out per server, or a dedicated ID-generation service) avoids turning the counter itself into a single point of contention.
  - Caching the hot subset of short-code-to-URL mappings close to the edge is what keeps redirect latency low, since the long tail of links is rarely read but must still be stored durably.
  - A 301 redirect is cacheable by browsers and CDNs but hides future click analytics from your service, while a 302 keeps every click visible at the cost of repeat server round-trips.
  - The write path (shortening a URL) and the read path (resolving a redirect) have such different load profiles that they're often worth scaling as separately as possible.
check_questions:
  - q: Why does base62-encoding a counter avoid the collision-retry loop that hashing a URL requires?
    a: A counter only ever increments, so each value it produces is guaranteed unique before it's even encoded, whereas a hash of the URL (like the first few characters of MD5) can collide with an existing code and force a check-and-retry step before the write succeeds.
  - q: What's the main risk of using a single auto-increment counter in a database for short-code generation, and how is it usually solved?
    a: A single counter becomes a write bottleneck and single point of failure under high shortening traffic, so most designs hand out non-overlapping ranges of counter values to each application server (or a dedicated ID-generation service) so servers can mint codes locally without coordinating on every request.
  - q: Why is caching more important for this system's read path than its write path?
    a: Reads (redirects) outnumber writes (new short URLs created) by a large margin in essentially every real deployment, and a cache hit on the hot set of popular links turns a redirect into a sub-millisecond in-memory lookup instead of a database round trip, which matters because users feel redirect latency directly.
deeper:
  - label: "How Does a URL Shortener Work? (ByteByteGo)"
    url: "https://www.youtube.com/watch?v=HHUi8F_qAXM"
related_formulas: []
---

## Requirements

Start by pinning down what "design a URL shortener" actually means, since the phrase hides a lot of optional scope.

Functional requirements:
- Given a long URL, return a short one (e.g., `short.ly/aZ3kQ9`).
- Given a short URL, redirect the client to the original long URL.
- Optionally: let a user pick a custom alias, set an expiration date, and see click analytics.

Non-functional requirements, which end up driving almost every interesting decision:
- **Read-heavy**: redirects vastly outnumber new-link creations — real-world ratios are commonly quoted around 100:1 or higher.
- **Low redirect latency**: a redirect sits directly in a user's click path, so every extra millisecond is felt.
- **High availability over strong consistency**: a short link resolving to the wrong URL is bad, but a short redirect service being briefly unavailable is worse for user trust, so this system leans available.
- **Uniqueness**: two different long URLs must never collide on the same short code (collisions in the other direction — the same long URL getting two different short codes — are usually fine).

## Capacity estimate

Suppose the service targets 100 million new short URLs per month and a 100:1 read-to-write ratio.

- Writes: 100M / month ≈ 40 URLs/second average, say 200-400/second at peak.
- Reads: 100:1 ratio ≈ 4,000 redirects/second average, several times that at peak.
- Storage: if each stored record (short code, long URL, metadata) is roughly 500 bytes, 100M/month × 500 bytes ≈ 50 GB/month, or ~3 TB over 5 years — small enough that storage cost is a non-issue; the real constraints are throughput and latency, not capacity.
- Short code length: base62 (a-z, A-Z, 0-9) with 7 characters gives 62^7 ≈ 3.5 trillion possible codes — comfortably enough for years of growth at this write rate without reusing codes.

The takeaway from the back-of-the-envelope pass: this is not a storage problem, it's a *read-latency-at-scale* problem.

## High-level design

A client's "shorten" request hits a load balancer, which routes to a stateless application server. That server asks an ID-generation step for a unique number, base62-encodes it into a short code, writes `{short_code → long_url}` into the primary datastore, and returns the short URL.

A client's redirect request (`GET short.ly/aZ3kQ9`) also hits the load balancer and an application server, but that server first checks a distributed cache for the short code. On a cache hit, it issues the HTTP redirect immediately. On a miss, it reads from the datastore, populates the cache, and then redirects. The datastore itself is a simple key-value lookup — a relational table with an index on the short code works fine, or a key-value/NoSQL store if the team wants to avoid relational overhead for what's structurally a single-key lookup (see [SQL vs. NoSQL and data modeling](/roadmaps/system-design/sql-vs-nosql-and-data-modeling)). Because reads and writes have such different profiles, it's common to let the read path scale independently — more cache nodes and more read replicas — without scaling the write path at the same rate.

## Key decisions

**1. How short codes get generated.** Two broad approaches compete here. The first is a globally unique counter, base62-encoded: simple, collision-free by construction, and fast, but it requires either a single coordinating counter (a scaling bottleneck) or pre-allocated ranges handed out to each server so they can encode locally without talking to each other on every request. The second is hashing the long URL (e.g., take the first 7 characters of its MD5 hash) and using that as the code: this avoids any shared counter, but two different URLs can hash to overlapping prefixes, so the write path needs a check-and-retry loop (hash, check if taken, append a salt and rehash if so) before it can commit. In practice, the counter-plus-ranges approach is usually preferred precisely because it sidesteps that retry loop entirely — it trades a small amount of up-front coordination (handing out ranges) for guaranteed-unique output on every single write, which matters more as write volume grows.

**2. The read-heavy caching strategy.** Given the 100:1-or-higher read skew, the redirect path is the one worth optimizing hardest. A cache-aside pattern — check the cache, fall through to the datastore on a miss, populate the cache, return — paired with an LRU-style eviction policy handles the typical "most clicks concentrate on a small, recently-created subset of links" access pattern well (see [distributed caching systems](/roadmaps/system-design/distributed-caching-systems)). The harder question is where that cache lives: a shared distributed cache (Redis/Memcached cluster) in front of the datastore cuts database load company-wide, while pushing cacheable, well-known-TTL redirects out to a CDN edge cuts latency further still for geographically distributed users, at the cost of losing per-click visibility at the origin.

**3. Redirect status code, and what it costs you.** A `301 Moved Permanently` tells browsers and CDNs "cache this forever," which is great for load — the client may never hit your server again for that link — but terrible for analytics, since you lose the ability to count every subsequent click. A `302 Found` tells clients "this mapping might change, ask me again next time," which keeps every click flowing through your service (useful if click tracking or link expiration is a real requirement) at the cost of that traffic never getting offloaded to a browser or CDN cache. This is a genuine product tradeoff, not just a technical one — a link-shortening product that sells click analytics almost always has to pick 302 and eat the extra load.

## Tradeoff and failure mode worth calling out

The single most consequential tradeoff is **counter range size vs. blast radius on server crash**. If each application server is handed a large pre-allocated range of counter values (say, a million at a time) to minimize coordination overhead with the central counter service, then a server that crashes mid-range leaves those unused values permanently burned — never reclaimed, just gone. At 3.5 trillion possible codes this is harmless in isolation, but it compounds: a fleet that churns servers frequently (autoscaling up and down under load) can burn through ranges far faster than raw write volume alone would suggest, quietly eating into the keyspace. The fix isn't to eliminate the waste — reclaiming partial ranges safely is its own hard coordination problem — it's to size ranges so that even a worst-case churn rate, sustained for the system's expected lifetime, doesn't meaningfully threaten keyspace exhaustion, and to monitor consumption rate as a leading indicator rather than discovering the problem only when codes run out.
