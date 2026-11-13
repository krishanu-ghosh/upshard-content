---
title: "API Architectural Styles: REST, GraphQL, and gRPC"
description: What REST, GraphQL, and gRPC each optimize for, and how to pick between them for a given API.
roadmap: system-design
node: api-architectural-styles
group: networking-apis-and-communication
video:
  provider: youtube
  id: "4vLxWqE94l4"
  start: 0
  end: null
creator:
  name: ByteByteGo
  channel_url: https://www.youtube.com/@ByteByteGo
duration_seconds: 260
level: intermediate
prerequisites:
  - message-queues-and-async-communication
takeaways:
  - REST models an API as a set of resources addressed by URLs, manipulated with standard HTTP verbs, and is stateless — each request carries everything the server needs to handle it.
  - REST's biggest practical weakness is over-fetching and under-fetching — a fixed response shape either returns more data than a client needs or forces several round trips to assemble what it needs.
  - GraphQL lets the client specify exactly which fields it wants in a single request to a single endpoint, which directly solves REST's over/under-fetching problem at the cost of a more complex server implementation.
  - gRPC uses a compact binary format (Protocol Buffers) and HTTP/2, trading human-readability for speed and strongly-typed contracts, which makes it a common choice for service-to-service calls inside a backend rather than for public-facing browser APIs.
  - A cloud design pattern like an API gateway or a circuit breaker sits around any of these styles — it doesn't replace REST, GraphQL, or gRPC, it manages traffic and failure handling for whichever style a system uses.
check_questions:
  - q: What does it mean for REST to be "stateless," and why does that matter at scale?
    a: Statelessness means every request contains all the information the server needs to process it, with no reliance on data saved from a previous request; that matters at scale because any server in a pool can handle any request, which makes horizontal scaling and load balancing straightforward since there's no session affinity to manage.
  - q: What's a concrete example of REST's over-fetching problem?
    a: A mobile app that only needs a user's name and avatar for a header might still get a full REST user object back — including address, settings, and order history — because the endpoint returns the whole resource; the client downloads and discards data it never needed.
  - q: Why is gRPC a more common choice between internal microservices than for a public API consumed by a web browser?
    a: gRPC's binary Protocol Buffers format and strict typed contracts are fast and efficient for services that both sides control and can update together, but that same binary format isn't natively readable by a browser or easy to debug by hand the way JSON over REST is, which makes REST or GraphQL the more practical choice at the public, browser-facing edge.
deeper:
  - label: "REST v GraphQL v gRPC | API Design"
    url: "https://www.youtube.com/watch?v=Rlv54zos8uc"
related_formulas: []
---

## Three answers to "how should services talk to each other"

Once two systems need to exchange data over a network — a browser and a backend, or two backend services — something has to define the shape of that conversation: what a request looks like, what a response looks like, and what guarantees either side can rely on. REST, GraphQL, and gRPC are three different, widely-used answers to that question, and each one optimizes for a different set of priorities.

## REST: resources and verbs

**REST** (Representational State Transfer) models an API as a collection of **resources** — a user, an order, a product — each addressed by its own URL, and manipulated using standard HTTP verbs: `GET` to read, `POST` to create, `PUT` or `PATCH` to update, `DELETE` to remove. A request to `GET /users/42` fetches user 42; a request to `POST /orders` creates a new order. This mapping is intuitive, it reuses HTTP's existing semantics (status codes, caching headers, verbs) instead of inventing new ones, and it's stateless — every request is self-contained and carries everything the server needs, with nothing remembered between requests. Statelessness is what makes REST so easy to scale horizontally: any server behind a load balancer can handle any request, because there's no server-side session state tying a client to a specific machine.

REST's weakness shows up around the *shape* of its responses. A single endpoint returns a fixed structure, so a client that only needs two fields from a large object still downloads the whole thing — **over-fetching**. Conversely, a client that needs data spread across several resources (a user, their orders, and each order's line items) often has to make several round trips to separate endpoints to assemble it — **under-fetching**. Neither problem is fatal, but both get worse as an API serves a growing variety of clients with different needs from the same endpoints.

## GraphQL: let the client ask for exactly what it needs

**GraphQL** addresses over- and under-fetching directly by inverting who decides the response shape. Instead of many fixed endpoints, a GraphQL API exposes a single endpoint and a schema describing all the data available; the client sends a query specifying exactly which fields it wants, potentially pulling from several related objects in one request, and the server returns exactly that shape — nothing more, nothing less.

This solves the mobile-header example cleanly: a client that only needs a name and an avatar asks for just those two fields, in one request, regardless of how large the full user object is or how many other resources it's related to. The tradeoff is on the server side: implementing a GraphQL API means building a schema and resolvers that can satisfy flexible, client-specified queries, which is more upfront work than exposing a fixed REST endpoint, and it also makes server-side caching and rate-limiting harder, precisely because the shape of each request isn't fixed in advance the way a REST URL is.

## gRPC: speed and strict contracts for service-to-service calls

**gRPC** takes a different path entirely. Rather than optimizing for flexibility or human-readability, it optimizes for performance and strict correctness between systems that are built and deployed together. It uses **Protocol Buffers** — a compact binary serialization format — instead of human-readable JSON, and it runs over HTTP/2, which supports multiplexed requests and native streaming (a client and server can exchange a continuous stream of messages, not just one request and one response).

The cost of that efficiency is accessibility: a binary payload isn't something you can read in a browser's network tab or curl by hand the way you can a REST or GraphQL response, and both sides need to share a compiled definition of the message formats (the `.proto` file) ahead of time. That tradeoff makes gRPC a strong fit for internal service-to-service communication inside a backend — where both sides are under your control, performance matters because calls happen constantly, and strict typed contracts catch mismatches early — and a poor fit for a public API consumed directly by a web browser or an external developer who has no reason to adopt your binary format.

## None of this is the whole picture

Choosing REST, GraphQL, or gRPC answers the question "what does a single request and response look like," but it doesn't address everything a real system needs around that traffic — routing requests to the right service, retrying or failing gracefully when something downstream is unhealthy, enforcing rate limits, or terminating TLS in one place instead of every service. Those concerns are handled by cloud design patterns that sit *around* whichever API style you've chosen: an **API gateway** acts as a single entry point that can route to multiple backend services regardless of their internal style, and a **circuit breaker** stops a client from repeatedly calling a service that's already failing, giving it time to recover instead of piling on more load. The architectural style decides the shape of a conversation; these patterns decide how traffic and failure are managed around it.
