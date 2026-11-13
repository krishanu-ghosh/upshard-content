---
title: "DNS and Request Routing at Scale"
description: How anycast and GeoDNS turn DNS into a coarse load balancer, and the TTL tradeoff between fast failover and cache efficiency.
roadmap: system-design
node: dns-and-request-routing-at-scale
group: networking-apis-and-communication
video:
  provider: youtube
  id: "-2WnHiKPrgE"
  start: 0
  end: null
creator:
  name: F5 DevCentral Community
  channel_url: https://www.youtube.com/@devcentral
duration_seconds: 382
level: intermediate
prerequisites:
  - back-of-the-envelope-estimation
takeaways:
  - DNS can be used as a coarse-grained load balancer by handing different clients different IP addresses for the same hostname, shifting traffic before a single packet reaches a data center.
  - Anycast announces the same IP address from multiple physical locations and lets internet routing (BGP) deliver each packet to a topologically nearby location, with failover measured in the seconds it takes routes to reconverge.
  - GeoDNS and latency-based routing work one layer up, at the DNS resolver, answering each query with a different IP based on the querying resolver's location or measured latency to each candidate region.
  - A DNS record's TTL is a dial between fast failover and cache efficiency — a short TTL lets you redirect traffic quickly when a region fails, but it forces resolvers to re-query far more often, adding load and lookup latency.
  - Anycast reacts to network-layer routing changes and ignores server health, while DNS-based routing can factor in health checks and load but only controls which address a resolver receives, not the packet path — production systems often combine both.
check_questions:
  - q: Why is anycast's failover faster than a typical DNS-based failover?
    a: Anycast failover happens at the routing layer — once the unhealthy location withdraws its BGP announcement, routers worldwide reconverge on the next-nearest location within seconds, with no dependency on any client's cached DNS answer expiring first.
  - q: What's the practical downside of setting a DNS record's TTL very low to enable fast failover?
    a: A low TTL means resolvers can't cache the answer for long, so they re-query the authoritative DNS servers far more often, which adds load to those DNS servers and adds a DNS lookup's worth of latency to a larger fraction of requests.
  - q: Why might a system use both anycast and GeoDNS rather than just one?
    a: Anycast is excellent for getting packets to a nearby location fast but can't see server load or health, while GeoDNS can incorporate health checks and load data but only shifts traffic as fast as TTLs expire — combining them gets fast, topology-aware routing at the network layer with health- and load-aware steering at the DNS layer.
deeper:
  - label: "How Google designed their Highly Available Load Balancers using Anycast"
    url: "https://www.youtube.com/watch?v=WjT253DBlXk"
  - label: "GEO DNS - Direct traffic based on origin geography"
    url: "https://www.youtube.com/watch?v=ad1q3PEZbNc"
related_formulas: []
---

## DNS isn't just a lookup table — it's a traffic-shaping tool

A system-design-fundamentals lesson elsewhere in this course already covers what DNS does: translate a hostname into an IP address. At scale, that same translation step becomes a lever for *where* traffic goes. If a DNS server can hand out different answers to different clients for the same hostname, it has effectively become a load balancer that acts before a single TCP packet is sent — no proxy, no data-center hop, just a different IP address returned to the resolver. Two distinct mechanisms get used for this, and they operate at different layers of the stack.

## Anycast: let routing pick the nearest copy

**Anycast** means announcing the *same* IP address from multiple physical locations — say, ten data centers around the world all claim to own `203.0.113.1`. Internet routers run BGP (Border Gateway Protocol) to figure out the shortest path to any given address, and because multiple locations announce the identical address, BGP's normal "shortest path" logic ends up sending each client's packets toward whichever announcing location is topologically closest to that client. A user in Tokyo and a user in São Paulo requesting the same anycast address are transparently routed to different physical servers, with no DNS trickery involved at all — it happens purely at the network layer.

The appealing property of anycast is failover speed. If the Tokyo location goes down, it simply stops announcing that route; routers worldwide reconverge on the next-nearest announcing location within the time it takes BGP routes to propagate — typically seconds. There's no cache to wait out, because there was never a DNS answer caching a *specific* server's address — the address was always shared. The tradeoff is that anycast is blind to server load and application health by default: it optimizes for network path length, not for which location has spare capacity or is actually serving requests correctly. Real deployments pair anycast with health checks that withdraw a route when something is actually broken, rather than relying on BGP alone.

## GeoDNS and latency-based routing: steer at the DNS layer instead

**GeoDNS** (and its cousin, latency-based routing) takes a different approach: instead of making every server share one address, each physical location gets its own distinct IP address, and the *authoritative DNS server* decides which address to hand back depending on who's asking. A GeoDNS server might look at the resolver's apparent location and answer a query from Germany with the Frankfurt data center's address, while answering the same hostname query from Virginia with the US-East address. Latency-based routing variants go further, maintaining measured or estimated latency from different network regions to each candidate location and picking whichever address is fastest for that resolver, rather than relying purely on geography (which can be a poor proxy — a resolver's location isn't always the client's location, and "closest" isn't always "fastest").

This gives you something anycast can't: the DNS server can factor in real signals like each location's current load or health-check status when deciding what to serve, not just network topology. The cost is latency to react. A DNS answer is cached by resolvers for as long as its **TTL** (time-to-live) says it's valid, and nothing forces an already-cached answer to update early — if a resolver cached the Frankfurt address with a one-hour TTL five minutes before Frankfurt went down, clients behind that resolver keep trying Frankfurt for up to the remaining 55 minutes.

## The TTL dial: fast failover versus cache efficiency

This is the central tradeoff of DNS-based routing: **TTL** trades failover speed against cache efficiency, directly.

- A **short TTL** (say, 30 seconds) means a bad answer gets flushed out of caches quickly, so you can redirect traffic away from a failing region almost as fast as you can update the DNS record. But it also means every resolver that would otherwise have cached the answer for an hour is now re-querying your authoritative DNS servers 120 times more often — more DNS query load, and every one of those re-queries adds a DNS lookup's worth of latency to whatever request triggered it.
- A **long TTL** (say, one hour) is cheap: resolvers rarely re-query, DNS infrastructure load stays low, and most requests skip the DNS round trip entirely by using a cached answer. But if something breaks, you're stuck waiting for every cached copy of the bad answer, scattered across resolvers you don't control, to expire naturally.

A system that needs to fail over in seconds — a global service with strict availability targets — generally accepts a short TTL (often 30–60 seconds) on the records that matter for routing, eating the extra query volume as the cost of being able to redirect quickly. A system serving mostly static, rarely-changing infrastructure can run a much longer TTL and skip that cost. There's no universally correct value; it's a dial set based on how fast you need to be able to move traffic versus how much DNS query volume and latency you're willing to pay for that speed.

## Combining the two layers

Because anycast reacts in seconds but is blind to load, and GeoDNS/latency routing can see load but reacts only as fast as its TTL allows, many large-scale systems use both: anycast gets packets to a nearby point of presence almost instantly and survives network-level outages gracefully, while DNS-based routing (with a carefully tuned TTL) steers traffic between regions based on health and load signals that anycast alone can't see. Neither mechanism replaces a reverse proxy or a load balancer sitting behind it — they're the first, coarsest routing decision a request encounters, well before it reaches any server that can inspect the request itself.
