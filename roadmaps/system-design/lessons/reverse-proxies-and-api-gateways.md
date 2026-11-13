---
title: "Reverse Proxies and API Gateways"
description: What a reverse proxy adds over a plain load balancer, how it differs from a forward proxy, and what an API gateway layers on top.
roadmap: system-design
node: reverse-proxies-and-api-gateways
group: load-balancing-and-service-discovery
video:
  provider: youtube
  id: "xo5V9g9joFs"
  start: 0
  end: null
creator:
  name: TechWorld with Nana
  channel_url: https://www.youtube.com/@TechWorldwithNana
duration_seconds: 798
level: intermediate
prerequisites:
  - layer4-vs-layer7-load-balancing
takeaways:
  - A forward proxy acts on behalf of clients and requires client-side configuration, while a reverse proxy acts on behalf of servers and is invisible to the client.
  - A reverse proxy's reason to exist is broader than a load balancer's — it can add TLS termination, compression, and response caching even in front of a single backend with nothing to balance.
  - TLS termination at a reverse proxy means certificates live in one place instead of on every backend, and backends can run on a network that never has to speak TLS at all.
  - Response caching at a reverse proxy means an identical request can be answered without the backend ever seeing it again, which is distinct from CDN caching and browser caching covered later in this roadmap.
  - An API gateway is a reverse proxy specialized for routing across multiple backend services, and it adds centralized authentication, rate limiting, and service routing on top of what a plain reverse proxy already does.
  - Each layer in a real request path — load balancer, reverse proxy, API gateway — is strictly more aware of the request's content than the layer below it, even though one piece of software often plays more than one of these roles at once.
check_questions:
  - q: What's the practical test for telling whether something is a forward proxy or a reverse proxy?
    a: If removing it would require reconfiguring the clients to stop pointing at it, it's a forward proxy; if removing it would require reconfiguring or directly exposing the servers instead, it's a reverse proxy.
  - q: Why would you run a reverse proxy in front of a single backend server that has nothing to load-balance between?
    a: Because a reverse proxy's role includes TLS termination, compression, and response caching, all of which are useful on their own regardless of how many backend instances exist, unlike a load balancer, whose only job is choosing between multiple backends.
  - q: What does an API gateway add on top of what a plain reverse proxy already does?
    a: It adds centralized authentication and authorization, rate limiting, and routing across multiple distinct backend services, rather than just adding connection-level concerns like TLS termination or caching in front of one service.
deeper:
  - label: "Reverse Proxy vs API Gateway vs Load Balancer — ByteByteGo"
    url: "https://www.youtube.com/watch?v=RqfaTIWc3LQ"
  - label: "What is API gateway really all about? — Java Brains"
    url: "https://www.youtube.com/watch?v=1vjOv_f9L8I"
related_formulas: []
---

## Forward proxy vs reverse proxy

A proxy sits between two parties and relays traffic on behalf of one of
them, and which side it represents is the whole distinction. A **forward
proxy** sits in front of clients and acts on their behalf: a corporate
network's outbound proxy, for instance, is a forward proxy — every
employee's browser is configured to send its traffic through it, so the
proxy can enforce a browsing policy, cache frequently-requested pages, or
anonymize the client's IP from the destination server's point of view. The
server on the other end usually doesn't know, or care, that a proxy is
involved; from its perspective, a request just arrived. The client, by
contrast, explicitly knows about and is configured to use the forward proxy.

A **reverse proxy** sits in front of servers and acts on their behalf, and
the awareness is flipped: clients send requests to what looks like "the
server" with no configuration change on their end, and the reverse proxy
decides, invisibly, which actual backend handles each request. This is the
role covered in the previous lesson's L4/L7 distinction — a reverse proxy is
typically an L7 device, since deciding "which backend" by path or header
requires reading the request. Nginx running in front of a fleet of
application servers, or a cloud load balancer terminating TLS before
forwarding to backend instances, is acting as a reverse proxy: from outside,
there's one address; behind it, there could be dozens of changing instances.

The practical test for which kind you're looking at: if removing the proxy
would require reconfiguring clients, it's a forward proxy. If removing it
would require reconfiguring servers (or just exposing them directly, which
you don't want to do), it's a reverse proxy.

## What a reverse proxy adds over a plain load balancer

A plain load balancer's job, as covered in the previous two lessons, is
choosing which backend gets a request. A reverse proxy typically does that
too, but the name "proxy" signals a broader role: it fully terminates the
connection and can rewrite, inspect, or respond to the request on its own,
not just forward it somewhere. That broader role usually includes:

- **TLS termination.** Certificates live at the proxy, not on every backend,
  and backends can run on an internal network that never speaks TLS
  directly.
- **Compression.** The proxy can gzip or brotli-compress a response before
  it goes out over the (often slower, less trusted) client-facing link, even
  if the backend never compressed anything.
- **Response caching.** If the proxy knows a response is cacheable (from
  `Cache-Control` headers, or static-asset rules), it can serve the next
  identical request itself, without the backend ever seeing a second
  request at all — this is distinct from a CDN, which does the same thing
  but geographically distributed, and from browser caching, which happens
  entirely client-side (both get their own lessons later in this roadmap).
- **Request/response rewriting.** Stripping an internal header before it
  reaches the client, adding an `X-Request-Id` for tracing, or normalizing a
  URL before it hits the backend.

None of this requires multiple backends — you could run a reverse proxy in
front of a single server purely for TLS termination and caching. That's the
real conceptual difference from a load balancer: a load balancer's reason to
exist is "there's more than one backend to choose between," while a reverse
proxy's reason to exist is "something useful should happen to this
request/response before it reaches the client or the backend," which is a
superset that happens to include load balancing as one option among several.

## Layering an API gateway on top

An API gateway is a reverse proxy purpose-built for APIs, usually in front
of multiple backend *services* rather than multiple interchangeable
*instances* of one service. It adds a layer of concerns that are specific to
serving APIs to many different kinds of clients:

- **Centralized authentication and authorization** — verifying a token or
  API key once, at the edge, instead of every backend service
  re-implementing the same check.
- **Rate limiting** — enforcing per-client or per-API-key request limits
  before traffic ever reaches a backend.
- **Routing to backend services** — not just "which instance of this one
  app," but "which of twenty different microservices should handle
  `/orders` vs `/payments` vs `/users`," often with request/response shape
  translation between what the client expects and what each service
  actually returns.

Both authentication and rate limiting get full lessons of their own later in
this roadmap, so the detail here is intentionally shallow — what matters now
is the shape of the layering: a load balancer picks an instance, a reverse
proxy adds connection-level and caching concerns on top of that, and an API
gateway adds a further layer of per-client, per-API policy on top of *that*.
Each layer is strictly more aware of what's actually being asked for than
the one below it.

## Where this fits in a real request path

A realistic edge for a service-oriented system might look like: DNS resolves
to an L4 load balancer, which spreads raw connections across a pool of L7
reverse proxies, each of which terminates TLS, checks a cache, and either
answers directly or forwards to an API gateway, which authenticates the
caller, applies a rate limit, and routes to the correct backend
microservice. In smaller systems several of these roles collapse into one
piece of software — Nginx or Envoy can act as reverse proxy and API gateway
at once — but the roles themselves don't disappear just because one process
plays more than one.
