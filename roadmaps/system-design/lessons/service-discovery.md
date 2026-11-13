---
title: "Service Discovery"
description: How services find each other at runtime — client-side vs server-side discovery, service registries, and health checks that keep them accurate.
roadmap: system-design
node: service-discovery
group: load-balancing-and-service-discovery
video:
  provider: youtube
  id: "GboiMJm6WlA"
  start: 0
  end: null
creator:
  name: Java Brains
  channel_url: https://www.youtube.com/@Java.Brains
duration_seconds: 479
level: intermediate
prerequisites:
  - reverse-proxies-and-api-gateways
takeaways:
  - Service discovery exists because instance addresses change continuously in dynamic systems, so a static, hardcoded list of backend addresses goes stale almost immediately.
  - In client-side discovery, the calling service queries the registry itself and picks an instance locally, which gives full control but requires a discovery-client library in every calling codebase.
  - In server-side discovery, the caller sends its request to a stable intermediary that queries the registry on its behalf, which simplifies every client but adds a network hop and a central point of coordination.
  - Tools like Consul, etcd, and ZooKeeper implement a registry as a consistent, replicated key-value store, usually backed by a consensus protocol such as Raft or ZAB.
  - Health checks remove instances that are registered but no longer healthy, which matters because a process can keep running while being functionally unable to serve requests.
  - Most container orchestrators build server-side discovery directly into their control plane, so a Kubernetes Service is this same pattern implemented as a platform feature rather than a separate registry.
check_questions:
  - q: Why does a hardcoded list of backend IP addresses stop working in a system with auto-scaling or frequently restarting containers?
    a: Instances come and go continuously as containers restart with new addresses or auto-scaling adds and removes them, so a hardcoded list goes stale almost immediately and needs something tracking the current set of healthy instances in real time instead.
  - q: What's the main cost of client-side service discovery compared to server-side discovery?
    a: Every calling service needs its own discovery-client library that knows how to query the registry and pick an instance, which means that logic is duplicated and must be kept consistent across every codebase that makes calls, in whatever language each one is written in.
  - q: Why do registries need health checks in addition to a registration mechanism?
    a: An instance can register itself and then crash, hang, or become unreachable without ever deregistering, and it can even keep its process technically running while being unable to serve requests, so the registry needs an active probe to detect and remove instances that are no longer actually healthy.
deeper:
  - label: "What is Service Discovery? — A Dev' Story"
    url: "https://www.youtube.com/watch?v=v4u7m2Im7ng"
related_formulas: []
---

## The problem: addresses that keep changing

Load balancers and reverse proxies, covered in the last two lessons, assume
someone already knows the pool of backend addresses to route across. Service
discovery is what builds and maintains that pool in the first place. In a
static system with a handful of long-lived servers, you could hardcode IP
addresses in a config file and call it done. That stops working the moment
instances are ephemeral — containers restart with new IPs, auto-scaling adds
and removes instances on demand, and a deployment might replace every
instance of a service within minutes. A hardcoded address list goes stale
continuously; something needs to track, in real time, which instances of a
service actually exist and are healthy right now.

Service discovery solves this with two cooperating pieces: a **registry**
that holds the current set of healthy instances for each service, and a
**discovery mechanism** that lets a caller turn "I need to talk to the
`orders` service" into an actual address. The interesting design question
is *where* that lookup happens — on the calling client, or on something in
front of it.

## Client-side discovery

In client-side discovery, the calling service itself queries the registry
directly, gets back a list of healthy instances for the target service, and
picks one — often using one of the load-balancing algorithms from earlier
in this group, like round robin or least connections, applied locally.
Netflix's early microservices stack (Eureka, paired with the Ribbon
client-side load balancer) is the canonical example of this pattern.

The advantage is that the caller has full control and visibility: it can
apply its own load-balancing logic, retry against a different instance
immediately on failure without a round trip to anything else, and avoid
adding an extra network hop through an intermediary. The cost is that every
service that calls other services needs a discovery-client library baked
in, in whatever language it's written in, and that library's behavior
(retry logic, caching of the instance list, how it reacts to a registry
outage) is now duplicated across every codebase in the system.

## Server-side discovery

In server-side discovery, the caller doesn't talk to the registry at all —
it sends its request to a well-known address (often a load balancer or
reverse proxy), and *that* intermediary queries the registry and routes to
a healthy instance on the caller's behalf. A cloud load balancer combined
with a container orchestrator's service registration is a common real-world
instance of this pattern: a caller just hits the load balancer's stable DNS
name, and the load balancer handles finding and routing to live instances
behind it.

This removes the discovery-client requirement from every calling service —
any client, in any language, that can make an HTTP request can use the
system, since all the discovery logic is centralized in the routing layer.
The tradeoff is an extra network hop on every call, and a new single point
of coordination: if the routing layer's view of the registry is stale or the
layer itself is overloaded, every caller behind it is affected at once,
rather than the failure being isolated to whichever clients happen to be
using a particular stale cache.

## Service registries and how they stay accurate

Whichever pattern is used, the registry itself needs two things: a way for
instances to register themselves (or be registered by an orchestrator) when
they come up, and a way to remove instances that are no longer healthy.
Tools like **Consul**, **etcd**, and **ZooKeeper** are the common building
blocks here, and they share a similar shape even though their APIs differ: a
consistent, replicated key-value store (often backed by a consensus
protocol — Raft for Consul and etcd, ZAB for ZooKeeper) holds the current
registration data, and clients can watch for changes instead of polling.

Registration alone isn't enough, because a process can register itself and
then crash, hang, or lose network connectivity without ever deregistering.
This is what **health checks** are for: the registry (or an agent next to
each instance) periodically probes each registered instance — hitting a
lightweight `/health` endpoint, or just checking that a heartbeat keeps
arriving on schedule — and removes any instance that fails enough
consecutive checks from the pool of addresses handed out to callers. A
service can be "up" in the sense that its process is still running while
being functionally dead for callers' purposes (database connection pool
exhausted, out of memory, deadlocked), which is exactly why health checks
probe actual readiness rather than just checking the process is alive.

## Choosing between the two patterns

Most systems today default toward server-side discovery, because it keeps
every calling service simple and language-agnostic, and modern container
orchestrators build service discovery into their own control plane rather
than requiring a separate registry at all — a Kubernetes `Service` object
is, under the hood, exactly this pattern: a stable name and address that
quietly resolves to whichever pod IPs are currently healthy. Client-side
discovery remains attractive when the extra hop's latency genuinely
matters, or when different callers legitimately need different routing
logic that a shared intermediary can't express — but it comes at the cost
of every client needing to speak the registry's protocol directly.
