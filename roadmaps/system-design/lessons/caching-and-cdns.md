---
title: "Caching Layers and CDNs"
description: Where caching sits in a system — browser, CDN, reverse proxy, application, database — and how a CDN serves content from the edge.
roadmap: system-design
node: caching-and-cdns
group: caching-at-scale
video:
  provider: youtube
  id: "dGAgxozNWFE"
  start: 0
  end: null
creator:
  name: ByteByteGo
  channel_url: https://www.youtube.com/@ByteByteGo
duration_seconds: 348
level: intermediate
prerequisites:
  - scaling-load-balancing-and-clustering
takeaways:
  - A system typically has several caching layers stacked on top of each other — browser, CDN, reverse proxy, application, and database — and a request can be satisfied by any of them without reaching the origin server.
  - The closer a cache sits to the user, the faster a hit is served and the more load it removes from everything behind it, which is why a CDN caching at the edge is so valuable for static content.
  - A cache hit skips the expensive work entirely; a cache miss falls through to the next layer and, eventually, has to do the real work and populate the caches on the way back.
  - A CDN is a network of geographically distributed servers that cache content close to users, so a request for a static asset doesn't have to cross the planet to reach the origin.
  - The eviction policy inside any one of these caches — LRU, LFU, or another — is what decides which entries survive when the cache fills up, whether that cache lives in a browser, a CDN edge node, or an application server's memory.
check_questions:
  - q: Why does a system usually have multiple caching layers instead of just one big cache?
    a: Each layer sits at a different distance from the user and protects a different part of the system — a browser cache avoids a network round trip entirely, a CDN avoids crossing the network to the origin, and an application cache avoids an expensive database query — so stacking them removes load at every stage instead of only the last one.
  - q: What makes a CDN effective for serving static assets like images and JavaScript bundles?
    a: A CDN replicates content across servers distributed close to users geographically, so a request is served from a nearby edge node instead of traveling to a single origin server, cutting both latency and the load the origin has to handle.
  - q: If a cache hit rate is low, is adding more cache capacity always the fix?
    a: Not necessarily — a low hit rate can also mean the eviction policy doesn't match the access pattern (for example LRU struggling with a workload that has frequently-reused old items), so the eviction policy and what gets cached matter as much as how much space the cache has.
deeper:
  - label: "What Is A CDN? How Does It Work?"
    url: "https://www.youtube.com/watch?v=RI9np1LWzqw"
related_formulas: []
---

## A request passes through several caches before it does real work

When a browser asks for a web page, the request doesn't go straight to the origin server and back. In a well-built system, it passes through a stack of caching layers first, and at any point along that stack the request might be answered without the next layer ever being bothered:

1. **Browser cache** — the browser itself may already have the asset stored locally from a previous visit, in which case the request never leaves the device.
2. **CDN** — if the browser does make a request, it's often routed to the nearest edge server of a content delivery network, which may already have a copy.
3. **Reverse proxy** — in front of the application servers, a reverse proxy (like Nginx or Varnish) can cache full responses for requests it's seen recently.
4. **Application cache** — inside the application itself, an in-memory cache (like Redis or Memcached) can hold the results of expensive computations or common queries.
5. **Database** — and only if nothing above had an answer does the request reach the database, which usually has its own internal caching for frequently-accessed pages of data.

Each layer that answers the request is one less layer of work done, and the layers closer to the user are also the cheapest and fastest to check. A hit at the browser costs nothing. A hit at the CDN costs one network round trip to a nearby server. A miss everywhere, falling all the way to the database, is the most expensive path a request can take — which is exactly why every layer above the database exists: to keep that expensive path rare.

## CDNs: caching at the edge

A **content delivery network** is a set of servers spread across many physical locations — "points of presence" — positioned close to where users actually are, rather than concentrated at one data center. When a user requests a static asset (an image, a CSS file, a video), the CDN routes that request to the nearest point of presence instead of the origin server.

The first time a given point of presence sees a request for an asset, it's a **cache miss**: the CDN node fetches the asset from the origin, serves it to the user, and stores a copy for next time. Every subsequent request for that same asset from a nearby user is a **cache hit**, served directly from the edge node with no trip back to the origin at all. The benefit compounds: latency drops because the physical distance the data travels shrinks, and the origin server's load drops because it's no longer serving every single request for popular content — just the first one per region, until the cache entry expires or gets evicted.

CDNs work best for content that's the same for everyone and doesn't change often — images, videos, JavaScript and CSS bundles, whole static pages. Content that's personalized per user or that must always be fresh (a live stock price, an account balance) is a worse fit, though CDNs increasingly support short cache lifetimes and edge logic to handle semi-dynamic content too.

## Reverse proxies: caching in front of the application

A **reverse proxy** sits between the internet and the application servers, and besides routing and load-balancing duties, it can cache full HTTP responses. If a reverse proxy has recently served a response for a given URL and the response is marked cacheable, it can return that same cached response to the next identical request without the application server ever being invoked. This is the same idea as a CDN, just one hop closer to the application rather than distributed geographically — useful when a CDN isn't in the picture, or as an additional layer behind one.

## Why eviction policy still matters at every layer

None of these caches has infinite space. A browser cache, a CDN edge node, a reverse proxy, and an application cache all eventually fill up and have to decide what to throw away to make room for something new — which is exactly the eviction-policy problem the previous lesson covered with LRU, LFU, and similar strategies. A CDN edge node running least-recently-used eviction on a popular asset that suddenly gets no traffic will drop it in favor of whatever's hot now; an application cache running the same policy on database query results does the exact same job, just at a different layer and a different timescale.

Seen this way, "caching" isn't one technique applied once — it's the same idea (remember the answer to avoid redoing the work) applied repeatedly at every layer a request passes through, each layer closer to the user and each one buying back latency and load from everything behind it.
