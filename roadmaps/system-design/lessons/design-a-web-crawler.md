---
title: "Case Study: Design a Web Crawler"
description: A worked design for a large-scale web crawler, covering the URL frontier, dedup, per-domain politeness, and the graph's unbounded size.
roadmap: system-design
node: design-a-web-crawler
group: case-studies
video:
  provider: youtube
  id: "BKZxZwUgL3Y"
  start: 0
  end: null
creator:
  name: Tech Dummies - Narendra Lakshmana Gowda
  channel_url: https://www.youtube.com/@TechDummiesNarendraL
duration_seconds: 2761
level: advanced
prerequisites:
  - backpressure-and-flow-control
  - distributed-consensus-paxos-and-raft
takeaways:
  - A web crawler never runs out of work by design, so "handle unbounded input gracefully" is a core requirement, not an edge case to patch in later.
  - The URL frontier — the queue of URLs waiting to be crawled — has to be prioritized and partitioned, not just a flat FIFO, or the crawler either starves important sites or hammers one domain.
  - Per-domain politeness (rate limits, respecting robots.txt, a minimum delay between requests to the same host) exists to protect the sites being crawled, and violating it gets a crawler blocked outright.
  - Deduplicating URLs at the scale of tens of billions of pages requires a space-efficient structure like a Bloom filter, because an exact set would consume more memory than is practical to keep fully in memory.
  - The crawl graph has no natural edge, so the crawler's own backpressure and prioritization logic is what keeps it from drowning itself in an ever-expanding frontier of discovered-but-not-yet-fetched URLs.
  - Freshness (recrawling a page that changes) and coverage (discovering new pages) compete for the same finite crawling capacity, and a mature crawler allocates budget between them deliberately rather than treating "crawl everything, then recrawl everything" as a single undifferentiated queue.
check_questions:
  - q: Why does a single flat FIFO queue work poorly as the URL frontier for a large-scale crawler?
    a: A flat FIFO treats every discovered URL as equally urgent and processes whichever domain happens to dominate the queue first, which both risks hammering one domain with a burst of requests (violating politeness) and lets a prolific but low-value domain crowd out URLs from higher-priority or less-represented domains, so the frontier needs per-domain queues and a priority scheme instead.
  - q: Why is a Bloom filter typically used for crawl deduplication instead of an exact set like a hash set?
    a: Tracking tens of billions of seen URLs exactly would require more memory than is practical to hold in memory (or would force slow disk-backed lookups), while a Bloom filter represents the same seen-set in a small fixed amount of memory with no false negatives and a small, tunable false-positive rate, trading a rare, acceptable re-crawl of an already-seen URL for a massive memory saving.
  - q: Why does crawling a domain too aggressively risk the crawler getting blocked entirely?
    a: Sending requests to one domain faster than its robots.txt-declared or reasonably inferred rate limit allows looks identical to a denial-of-service attack from the target site's perspective, and sites commonly respond to that pattern by blocking the crawler's IP range outright, which would cut off access to that domain's content far more completely than a modest, polite crawl rate ever would have cost in time.
deeper:
  - label: "Design a Web Crawler: FAANG Interview Question (ByteByteGo)"
    url: "https://www.youtube.com/watch?v=6u25GckPhLU"
related_formulas: []
---

## Requirements

A web crawler systematically discovers and fetches pages across the web (or a bounded subset of it), typically to feed a search index, an archive, or a data pipeline. The defining challenge isn't any single fetch — it's operating correctly against an input space that has no natural end.

Functional requirements:
- Starting from a set of seed URLs, discover linked pages and fetch their content.
- Avoid refetching the same URL redundantly within some time window.
- Respect each site's `robots.txt` and crawl-rate expectations.
- Periodically recrawl pages to detect changes (freshness), alongside discovering genuinely new pages (coverage).

Non-functional requirements:
- **Scalability**: the crawler needs to run across many machines fetching in parallel, since a single machine can't come close to covering a meaningful slice of the web in a reasonable time.
- **Politeness**: the crawler must not overload any individual site it crawls, which is both an ethical obligation and a practical necessity — getting blocked ends the crawl for that domain.
- **Extensibility and robustness**: the crawler will hit malformed HTML, infinite redirect loops, deliberately crawler-hostile pages, and dead links constantly, and none of it should take down the crawling fleet.
- **Freshness vs. coverage balance**: finite crawling capacity has to be split between finding new content and noticing that old content changed.

## Capacity estimate

Suppose the target is to crawl 10 billion pages with an average page size of 100 KB, aiming to refresh the whole set roughly every 30 days.

- Total fetches needed per cycle: 10 billion pages / 30 days ≈ 333 million pages/day ≈ ~3,900 pages/second sustained — and crawlers typically run well above this to leave headroom for retries, redirects, and failed fetches that don't count toward progress.
- Bandwidth: 3,900 pages/second × 100 KB ≈ 390 MB/second sustained just for page bodies, before accounting for DNS lookups, redirects, and robots.txt fetches, which adds real but smaller overhead.
- Frontier size: at any given moment the set of "discovered but not yet fetched" URLs can balloon far larger than the target page count, since a modestly sized crawl of the live web routinely discovers several outbound links per page — the frontier needs to be treated as its own large, persistent, prioritized store, not an in-memory queue.
- Dedup set: tracking 10+ billion seen URLs with a target false-positive rate of ~1% in a Bloom filter costs on the order of a few bytes per URL — tens of gigabytes total, comfortably shardable across the crawling fleet, versus the far larger footprint an exact set would need.

## High-level design

A **frontier manager** holds the queue of URLs waiting to be crawled, partitioned by domain (or host) so that per-domain politeness rules can be enforced independently and so that no single domain's queue can starve the others. A fleet of **fetcher workers** pulls URLs from the frontier — respecting each domain's configured delay between requests — downloads the page, and hands it to a **parser/extractor** stage that pulls out links and content. Extracted links get checked against a distributed **deduplication filter** (the Bloom filter) before being added back into the frontier, so already-seen URLs don't get re-queued. Fetched content flows onward to whatever consumes it — a search index builder, a content store — while a separate scheduling component periodically re-injects already-crawled URLs back into the frontier at a cadence based on how often that page has historically changed, balancing freshness against the capacity spent on first-time coverage. DNS resolution is typically cached and batched ahead of the fetchers, since a naive per-request DNS lookup would otherwise become its own bottleneck at this fetch rate (see [backpressure and flow control](/roadmaps/system-design/backpressure-and-flow-control) for how the frontier's growth is kept from outpacing the fetchers' ability to drain it).

## Key decisions

**1. The URL frontier and dedup at scale.** The frontier can't be a single FIFO — it needs a priority scheme (some pages are simply more valuable to crawl than others, by signals like inbound link count or historical change frequency) layered on top of per-domain partitioning (so the politeness policy for one domain can be enforced without any cross-domain coordination). Dedup is the companion problem: a URL that's already been crawled, or is already sitting in the frontier, shouldn't be re-added, and checking that at a scale of tens of billions of URLs is exactly the memory-vs-certainty tradeoff a Bloom filter is built for — a small, tunable false-positive rate (occasionally re-crawling something already seen) buys a dramatic reduction in memory versus an exact set.

**2. Politeness and per-domain rate limits.** This decision exists for a reason outside the crawler's own performance: hitting one domain too hard looks like an attack from that domain's point of view and risks getting the crawler's IP range blocked outright, which is a far worse outcome than crawling that domain slightly slower. Implementing this means every fetch has to be routed through a per-domain rate limiter (conceptually the same mechanism as the dedicated rate-limiter case study, just scoped per crawled domain rather than per API client — see [rate-limiting algorithms](/roadmaps/system-design/rate-limiting-algorithms)) and the frontier manager has to respect `robots.txt` crawl-delay directives per host, since ignoring them isn't just impolite, it's often an explicit violation of that site's stated terms for automated access.

**3. Handling the crawl graph's effectively unbounded size.** There's no natural stopping point — every crawl discovers more links than it fetches, and the live web keeps changing underneath a crawl that takes any real amount of time to run. The practical response is to treat the frontier as a bounded resource with explicit backpressure: when the frontier grows beyond a target size, the scheduler has to prioritize ruthlessly (crawl the highest-value unseen URLs first, let lower-priority discoveries wait or expire) rather than trying to eventually get to everything. Distributed coordination (via [consensus mechanisms](/roadmaps/system-design/distributed-consensus-paxos-and-raft) or a simpler leader-based partitioning scheme) keeps multiple frontier-manager instances from assigning the same domain's queue to two different worker pools at once, which would both double-fetch and double-violate that domain's politeness budget.

## Tradeoff and failure mode worth calling out

The tradeoff worth calling out is **crawler traps**: pages that generate an effectively infinite stream of distinct-looking URLs — calendar pages that link to "next month" forever, session IDs embedded in URLs that make every fetch look like a new page, or deliberately adversarial infinite-redirect chains. A naive crawler that simply follows every discovered link will pour unbounded resources into a trap, starving genuinely valuable crawling elsewhere in the frontier, and a Bloom filter alone doesn't save you if every trap-generated URL is syntactically unique. The practical defense is a combination of URL normalization (stripping session parameters, collapsing equivalent query-string orderings) to collapse what looks like infinite distinct URLs back down to the few that actually matter, plus a hard per-domain depth or fetch-count cap so that even an undetected trap can only consume a bounded, budgeted amount of crawling capacity before the frontier manager simply stops feeding it more work.
