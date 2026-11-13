---
title: "Case Study: Design a Search Autocomplete"
description: A worked design for a type-ahead search suggestion system, covering the trie structure, how suggestions stay fresh, and latency budget.
roadmap: system-design
node: design-a-search-autocomplete
group: case-studies
video:
  provider: youtube
  id: "us0qySiUsGU"
  start: 0
  end: null
creator:
  name: Tushar Roy - Coding Made Simple
  channel_url: https://www.youtube.com/@tusharroy2525
duration_seconds: 1182
level: advanced
prerequisites:
  - distributed-caching-systems
  - choosing-a-data-model
takeaways:
  - Search autocomplete is primarily a latency problem — suggestions have to appear within roughly 100-200 milliseconds of each keystroke, or the feature feels broken rather than helpful.
  - A trie is a natural fit for prefix lookup because every node on a path from the root already represents a shared prefix, making "give me everything starting with this prefix" a single subtree traversal instead of a search.
  - Storing a precomputed top-k list of popular completions at (or near) each trie node avoids re-ranking a potentially huge subtree on every single keystroke.
  - Precomputing top-k suggestions in an offline batch job keeps the read path extremely fast but means suggestions lag real query trends by however long that batch job's cycle takes.
  - A hybrid design — serve precomputed top-k results most of the time, with a lightweight, short-TTL adjustment layer for suddenly-trending queries — balances serving speed against freshness for breaking trends.
  - Because every keystroke can trigger a new request, client-side debouncing (waiting briefly after the last keystroke before firing a request) meaningfully cuts the request volume the backend has to serve.
check_questions:
  - q: Why is a trie a better fit for prefix-based autocomplete than a plain hash map of complete query strings?
    a: A hash map can only look up exact, complete keys, giving no way to efficiently find "all queries starting with this prefix" without scanning every entry, while a trie organizes strings by shared prefixes so that finding everything starting with a given prefix is just a matter of walking to that prefix's node and exploring its subtree.
  - q: Why do most autocomplete systems precompute the top-k suggestions per prefix rather than ranking candidates at request time?
    a: Ranking all possible completions under a popular short prefix at request time, for every keystroke of every user, would mean redoing a potentially large sorting or scoring computation well within a tight latency budget on every request, whereas precomputing and caching the top few results per prefix in an offline job turns the online read into a cheap, already-sorted lookup.
  - q: What's the tradeoff of relying entirely on offline, batch-precomputed top-k suggestions?
    a: Suggestions stay extremely fast to serve because the read path never does real-time ranking work, but the data only reflects query popularity as of the last batch run, so a suddenly-trending topic won't show up in suggestions until the next precomputation cycle completes, which is why many systems layer a smaller, fresher real-time adjustment on top of the batch-computed baseline.
deeper:
  - label: "Amazon interview question: Architecture for auto suggestions / type ahead (Tech Dummies)"
    url: "https://www.youtube.com/watch?v=xrYTjaK5QVM"
related_formulas: []
---

## Requirements

Search autocomplete (type-ahead) suggests likely completions as a user types into a search box — the feature behind Google's search bar, an e-commerce site's product search, or any search-heavy product's query box.

Functional requirements:
- Given a partial query (prefix), return a ranked list of the top-k likely completions.
- Suggestions should reflect overall query popularity, not just alphabetical matching.
- New or newly-popular queries should eventually surface as suggestions, not just a fixed, stale list.

Non-functional requirements:
- **Extremely low latency**: suggestions have to update, ideally, on every keystroke, which means a round trip budget on the order of 100-200 milliseconds end to end, leaving very little room for anything beyond a cache lookup.
- **High read throughput, read-heavy by an enormous margin**: essentially every keystroke in every search box is a read; actual searches (and the query-log writes that inform future suggestions) are comparatively rare.
- **Reasonable freshness**: suggestions don't need to be real-time to the second, but a system that takes weeks to reflect an obviously trending query feels broken.
- **Scalable storage for the underlying query-frequency data**: the raw query log driving suggestions can be enormous even if the served suggestion set per prefix is tiny.

## Capacity estimate

Suppose the product sees 1 billion searches/day, and each search session involves roughly 10 keystrokes that each trigger an autocomplete lookup (with reasonable client-side debouncing cutting what would otherwise be higher).

- Autocomplete read QPS: 1 billion × 10 / 86,400 seconds ≈ ~115,000 requests/second average, several times that at peak — this dwarfs the actual search-submission rate by roughly an order of magnitude, underscoring how read-heavy this system is.
- Each request should resolve in well under 100ms of backend time to leave headroom for network latency, so the serving path can afford essentially nothing beyond a cache or in-memory trie lookup — no database query, no real-time ranking computation.
- Query log volume feeding the ranking: 1 billion searches/day, even summarized to just (query text, count) pairs, is a dataset in the tens of gigabytes per day — large enough that it's processed in batch (e.g., a daily or hourly aggregation job), not read live on the request path.
- Serving-side memory footprint: a trie (or an equivalent prefix-indexed structure) over the top few million distinct query prefixes, each node holding a short precomputed top-k list, comfortably fits in a few gigabytes — small enough to be replicated in memory across many serving nodes rather than centralized.

## High-level design

An offline pipeline periodically aggregates the raw search query log into (query, frequency) counts, builds or updates a **trie** (or a similarly prefix-indexed structure) over those queries, and — critically — precomputes and attaches the top-k most frequent completions at or near each prefix node, so the online path never has to rank a subtree itself. That trie is then pushed out to a fleet of serving nodes, where it's held entirely in memory (or backed by a fast distributed cache) for low-latency lookups (see [distributed caching systems](/roadmaps/system-design/distributed-caching-systems)). On the read path, each keystroke's prefix is sent to a serving node, which walks the trie to the prefix's node and returns its precomputed top-k list directly — no ranking, no aggregation, just a lookup. A thin real-time layer can sit alongside this: recent search submissions get tallied in a fast, short-window counter, and a query that spikes unexpectedly can be boosted into suggestions before the next full offline rebuild completes, blending the batch-computed baseline with a lightweight freshness signal.

## Key decisions

**1. A trie (or similar structure) for prefix lookup.** The core requirement — "find everything that starts with this exact prefix, fast" — is precisely what a trie is built for: every path from the root spells out a shared prefix, so finding all completions of a prefix is just finding that prefix's node and walking its subtree, without scanning anything outside it. The main design choice within this is how much ranking information to store at each node: storing *just* the subtree structure means the read path still has to find and sort candidates in that subtree at request time, while storing a precomputed top-k list directly at (or compactly cached near) each relevant prefix node turns the read into a flat lookup — the standard choice once the latency budget is this tight. The cost is memory: storing a top-k list at every meaningful prefix node means some duplication of popular queries across many of their own prefixes, which is an acceptable trade given how cheap that storage is relative to the structure's read-latency payoff.

**2. How the structure is kept fresh.** Fully real-time ranking (recompute the top-k for a prefix on every request, from live query counts) would blow the latency budget outright and is essentially never done this way. Fully static, rarely-rebuilt precomputation is cheap to serve but can leave a genuinely trending query invisible for longer than is acceptable. The common middle ground is a two-speed system: a slower, heavier batch job (hourly or daily) rebuilds the authoritative trie from the full query log, while a smaller, much cheaper streaming layer tracks very recent query volume and can surface an unusually fast-rising query as a suggestion before it's earned its place in the next full rebuild — explicitly trading a small amount of serving-path complexity for materially better freshness on the cases that matter most (genuinely breaking or trending queries).

**3. The serving latency budget, and what it rules out.** Once the end-to-end budget is set at roughly 100-200ms including network time, the backend's actual processing time has to be a small fraction of that — which rules out essentially any database round trip or non-trivial computation on the request path. This is why the design pushes everything expensive (ranking, aggregation, building the structure) into the offline pipeline, leaving the online path doing nothing more than an in-memory structure walk, with the serving tier scaled out by simple replication (every node holds the same trie) rather than by partitioning, since the whole structure is small enough to fit comfortably in memory per node.

## Tradeoff and failure mode worth calling out

The tradeoff worth calling out is that **precomputing top-k per node trades staleness for speed in a way that's invisible until a real trend hits**. A design that serves only from the batch-rebuilt trie will look perfectly correct in everyday testing — common queries are stable, and the top-k lists barely change day to day — right up until a genuinely novel event drives a burst of searches for a term that didn't exist in the last batch run at all. Without the real-time adjustment layer, the system would simply show nothing useful for that term's prefixes until the next rebuild completes, which for a daily job could mean most of a day of a visibly broken-feeling feature during exactly the moment it would be most noticed. This is the general lesson behind most "precompute for speed" designs in search and recommendation systems: the precomputed path needs to be explicitly paired with a narrower, faster path for the specific failure mode precomputation creates, rather than assuming the batch cadence will always be fast enough to not matter.
