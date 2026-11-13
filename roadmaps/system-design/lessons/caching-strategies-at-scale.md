---
title: "Caching Strategies at Scale"
description: Cache-aside, write-through, write-behind, and refresh-ahead — how data actually gets into a cache and stays fresh, and when to use each.
roadmap: system-design
node: caching-strategies-at-scale
group: caching-at-scale
video:
  provider: youtube
  id: "rSQdrNzomN4"
  start: 0
  end: null
creator:
  name: JOY of LIFE
  channel_url: https://www.youtube.com/@joy_of_life
duration_seconds: 1552
level: intermediate
prerequisites:
  - caching-and-cdns
takeaways:
  - Cache-aside leaves the application in charge — it checks the cache first, and on a miss loads the data from the database itself and writes it into the cache, so the cache only ever holds what's actually been asked for.
  - Write-through writes to the cache and the database together, synchronously, as part of the same operation, which keeps the cache always consistent with the database at the cost of making every write wait on two systems instead of one.
  - Write-behind (write-back) writes to the cache immediately and returns, flushing the change to the database asynchronously afterward, which makes writes fast but risks losing unflushed data if the cache fails before the flush completes.
  - Refresh-ahead proactively reloads a cache entry before its TTL expires, based on a prediction that it'll be requested again soon, trading some wasted work on wrong predictions for avoiding a cache-miss latency spike on hot keys.
  - These four patterns answer "how does data get into the cache and stay fresh," which is a different question from eviction policy (what gets thrown out when the cache is full) — a system typically needs an answer to both.
check_questions:
  - q: What's the key difference between cache-aside and write-through when a write happens?
    a: In cache-aside, the application normally only populates the cache on a read miss, so a plain write often goes straight to the database and either invalidates or leaves the stale cache entry to be refreshed on the next read; in write-through, every write updates the cache and the database together as one synchronous operation, so the cache is never behind.
  - q: Why does write-behind improve write latency, and what's the risk it introduces?
    a: Write-behind returns to the caller as soon as the cache is updated, without waiting for the database write to happen, which makes the write fast from the caller's point of view; the risk is that if the cache crashes or loses data before the queued write reaches the database, that update is lost entirely, since the database was never actually told about it.
  - q: Why doesn't refresh-ahead replace the need for an eviction policy?
    a: Refresh-ahead only decides when to proactively reload an entry that's already in the cache and about to expire — it says nothing about what happens when the cache runs out of room for new entries, which is the separate problem eviction policies like LRU or LFU are designed to solve.
deeper:
  - label: "Distributed Caching Strategies: Cache-Aside, Write-Through, Write-Behind"
    url: "https://www.youtube.com/watch?v=V1oGVbxD8mk"
related_formulas: []
---

## A different question than eviction

The earlier lesson on caching layers and CDNs covered *where* caches sit in a system, and a sibling lesson covers eviction policies — what a cache throws away when it's full. This lesson is about a different, equally important question: when data is written, or when a cache entry is about to go stale, **how does the cache actually get populated and kept current?** The four patterns below — cache-aside, write-through, write-behind, and refresh-ahead — are the standard answers, and picking between them is really a decision about who's responsible for keeping the cache correct, and how much latency or risk you're willing to trade for speed.

## Cache-aside: the application does the work, lazily

**Cache-aside** (also called lazy loading) puts the application in charge of the cache explicitly. A read works like this:

```
value = cache.get(key)
if value is None:
    value = db.query(key)
    cache.set(key, value, ttl=300)
return value
```

The cache only ever contains what's actually been requested — nothing is pre-loaded speculatively. On a write, the simplest version of cache-aside just writes to the database and either deletes the now-stale cache entry or leaves it to expire via TTL, so the next read repopulates it with fresh data.

This is probably the most common pattern in practice because it's simple and resilient — if the cache is down entirely, reads just fall through to the database every time, degraded but correct. The cost is that every cache miss pays full database latency plus the cost of the cache write that follows it, and a cold cache (after a deploy or a cache-cluster restart) means a burst of misses all at once.

## Write-through: the cache is never behind

**Write-through** changes the write path: instead of writing only to the database and invalidating the cache, a write goes to the cache *and* the database as one synchronous step, and the operation isn't considered done until both have succeeded.

```
db.write(key, value)
cache.set(key, value)
# only now return success to the caller
```

The payoff is strong consistency between cache and database — a read immediately after a write-through write will always see the new value, with no invalidation window at all. The cost is write latency: every write now waits on two systems instead of one, and if the cache is temporarily unavailable, you have to decide whether to fail the write entirely or let it through to the database alone and risk the cache going briefly stale. Write-through fits workloads with frequent reads shortly after writes, where that read-after-write guarantee is worth paying for on every write.

## Write-behind: fast writes, asynchronous durability

**Write-behind** (or write-back) inverts write-through's ordering: the write lands in the cache first, the caller gets an immediate success response, and the database write happens afterward, asynchronously, often batched with other pending writes.

This makes writes about as fast as a cache write can be, and batching asynchronous flushes can also reduce database load — ten rapid writes to the same key might collapse into a single database write once they're flushed. The real risk is durability: between the moment the cache acknowledges the write and the moment it's actually flushed to the database, that data exists only in the cache. If the cache process crashes, or the node it's running on fails, before the flush happens, that write is gone — the database never heard about it, and there's no record anywhere else that it was ever attempted. Write-behind suits workloads that can tolerate that small durability window in exchange for low write latency and reduced database write pressure — session data or counters that update frequently and aren't individually critical are common fits; financial ledger entries generally are not.

## Refresh-ahead: don't wait for the miss

**Refresh-ahead** targets a specific pain point in TTL-based caching: a hot key that gets requested constantly will, the instant its TTL expires, force whichever request happens to hit it next to eat a full cache-miss round trip to the database — and under high concurrency, that can turn into many requests all missing at once (a cache stampede, covered in the next lesson). Refresh-ahead avoids this by proactively reloading an entry *before* it expires, typically when it's accessed within some threshold of its remaining TTL (say, requested with less than 20% of its TTL left), kicking off a background refresh rather than waiting for the entry to go stale.

```
value, remaining_ttl = cache.get_with_ttl(key)
if remaining_ttl < refresh_threshold:
    async_refresh(key)   # update the cache in the background
return value               # serve the (still valid) cached value now
```

The caller gets served instantly from the still-valid cache entry while the refresh happens behind the scenes, so no request pays the full miss penalty for a hot key. The tradeoff is wasted work when the prediction is wrong — refreshing a key that doesn't actually get requested again soon spends database and cache capacity on something nobody needed — so refresh-ahead works best applied selectively, to keys you have good reason to believe stay hot, rather than blanket-applied to everything in the cache.

## Choosing between them

These aren't mutually exclusive dogmas — real systems often mix them: cache-aside for most reads, write-through on a specific hot path that needs read-after-write consistency, refresh-ahead layered on top for a known set of hot keys. The decision criteria are consistent across all four: how much write latency can you tolerate, how much durability risk is acceptable between a write and its persistence, and how much you can predict about which keys will be hot next. None of this says anything about what happens when the cache fills up and something has to be evicted — that's the separate, complementary question the eviction-policies lesson answers.
