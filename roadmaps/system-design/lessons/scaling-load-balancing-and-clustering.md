---
title: "Scaling, Load Balancing, and Clustering"
description: Horizontal vs vertical scaling, how load balancers spread traffic across servers, and what clustering actually buys you.
roadmap: system-design
node: scaling-load-balancing-and-clustering
group: scaling-data-and-services
video:
  provider: youtube
  id: "dBmxNsS3BGE"
  start: 0
  end: null
creator:
  name: ByteByteGo
  channel_url: https://www.youtube.com/@ByteByteGo
duration_seconds: 318
level: intermediate
prerequisites:
  - replication-sharding-and-federation
takeaways:
  - Vertical scaling adds power to one machine (more CPU, RAM, faster disks), while horizontal scaling adds more machines and spreads the load between them.
  - Vertical scaling is simple but hits a hard ceiling and creates a single point of failure; horizontal scaling has no fixed ceiling but needs a way to split and route work.
  - A load balancer sits in front of a pool of servers and decides which one handles each incoming request, so clients never need to know how many servers exist behind it.
  - Round robin, least connections, and consistent hashing are different answers to "which server gets the next request," each suited to a different kind of workload.
  - A cluster is a group of servers that cooperate to appear as one logical system, usually with a mechanism to detect a failed member and keep serving without it.
check_questions:
  - q: Why does horizontal scaling usually win out over vertical scaling for a growing web service?
    a: Vertical scaling tops out at the biggest machine you can buy or rent and leaves a single point of failure, while horizontal scaling can keep adding commodity machines indefinitely and survives any one machine going down.
  - q: What's the practical difference between an L4 and an L7 load balancer?
    a: An L4 load balancer routes based on network-layer information like IP and port without looking at the request itself, while an L7 load balancer reads the actual HTTP request — path, headers, cookies — and can route on that content, at the cost of more processing per request.
  - q: Why might consistent hashing be a better routing strategy than round robin for a caching layer?
    a: Consistent hashing sends the same key to the same server as long as possible even as servers are added or removed, so cache hit rates stay high; round robin spreads requests evenly but with no regard for where a given key was last cached, which would spread identical lookups across every cache and waste most of the hits.
deeper:
  - label: "System Design BASICS: Horizontal vs. Vertical Scaling"
    url: "https://www.youtube.com/watch?v=xpDnVSmNFX0"
related_formulas: []
---

## Two ways to handle more load

Every system eventually runs into the same problem: it has to handle more traffic than it was built for. There are exactly two ways to respond. **Vertical scaling** means making the existing machine bigger — more CPU cores, more RAM, a faster disk, a beefier network card. **Horizontal scaling** means adding more machines and splitting the work between them.

Vertical scaling is appealing because it changes nothing about how the application is built — the code doesn't know or care that it's running on a bigger box. But it has two hard limits. First, there's a ceiling: at some point you're renting the largest instance a cloud provider sells, and there's nowhere left to go. Second, one machine is one machine — if it goes down, the whole service goes down with it, no matter how large that machine was.

Horizontal scaling has no fixed ceiling — in principle you can always add another server — but it demands something vertical scaling never had to: a way to decide which machine handles which piece of work, and a way for machines to cooperate as if they were one system. That's the problem load balancing and clustering solve.

## Load balancers: picking the next server

A load balancer sits between clients and a pool of backend servers. From the outside, a client just talks to "the service" at one address; the load balancer is the thing that actually picks which server behind that address handles each request, and the backend pool can grow, shrink, or have a member fail without the client ever noticing.

Load balancers operate at different layers of the network stack, and the layer determines how smart their routing can be:

- **Layer 4 (transport layer)** load balancers route based on IP address and port, without opening up the request itself. They're fast, because there's little to inspect, but they can't make decisions based on what the request actually contains.
- **Layer 7 (application layer)** load balancers read the full HTTP request — the path, headers, cookies, even the body — and can route based on that. A request to `/api/images` could go to a different pool of servers than `/api/search`, something an L4 balancer has no way to do.

Given a pool of eligible servers, the load balancer still needs an algorithm to pick one. A few common ones, each suited to different situations:

- **Round robin** cycles through servers in order, one request each. It's simple and works well when every request costs about the same amount of work and every server is equally capable.
- **Least connections** sends the next request to whichever server currently has the fewest active connections — a better fit when request costs vary a lot, since it naturally avoids piling more work onto a server that's already busy with long-running requests.
- **Consistent hashing** computes a hash of some property of the request (often a user ID or cache key) and uses that hash to pick a server, so the same input reliably lands on the same server. This matters enormously for anything with a cache behind it: if request routing is random, every server ends up caching the same popular keys redundantly instead of each server owning a slice of the keyspace.

None of these is universally "best" — a load balancer in front of a stateless API that does roughly the same amount of work per request is a natural fit for round robin, while a load balancer in front of a caching tier wants consistent hashing specifically because cache locality is the whole point.

## Clustering: many machines, one system

Clustering is the practice of configuring a group of servers to work together and present themselves as a single logical system, usually with built-in failure handling. A web server cluster behind a load balancer is the simplest example: any of the nodes can answer a request, and if one node crashes, the cluster keeps running on the rest while the load balancer stops sending it traffic.

Clustering gets more interesting — and harder — when the servers hold state, not just stateless request-handling logic. A database cluster, for instance, has to keep the data itself consistent across nodes, not just route requests to whichever node happens to be free; that's the territory covered by replication and sharding (see the previous lesson), which is why clustering for stateful systems always comes with an explicit strategy for how data is split or copied between members.

The payoff for getting clustering right is twofold: **availability** (the system tolerates individual machine failures without going down) and **capacity** (the system can keep absorbing more load by adding more machines to the cluster, which is exactly the horizontal-scaling story this lesson opened with). A load balancer and a cluster are two halves of the same idea — the load balancer decides where a request goes, and the cluster is the group of machines ready to receive it and keep working even when one of them fails.
