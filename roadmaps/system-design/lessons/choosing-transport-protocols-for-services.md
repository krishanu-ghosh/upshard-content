---
title: "Choosing Transport Protocols for Services"
description: TCP vs UDP for service-to-service calls, how HTTP/1.1 through HTTP/3 changed multiplexing, and why keep-alive matters at scale.
roadmap: system-design
node: choosing-transport-protocols-for-services
group: networking-apis-and-communication
video:
  provider: youtube
  id: "GriONb4EfPY"
  start: 0
  end: null
creator:
  name: Hussein Nasser
  channel_url: https://www.youtube.com/@hnasr
duration_seconds: 591
level: intermediate
prerequisites:
  - dns-and-request-routing-at-scale
takeaways:
  - TCP gives you ordered, reliable delivery and congestion control, which is why most service-to-service RPC defaults to it, but UDP's lack of those guarantees makes it the right choice when an application can tolerate loss and wants to avoid retransmission delay, as with real-time media or some telemetry.
  - HTTP/1.1 opens one request at a time per connection (or relies on multiple parallel connections to fake concurrency), so a backend making many small calls either serializes them or pays the cost of juggling several TCP connections.
  - HTTP/2 multiplexes many requests over a single TCP connection, which removes HTTP-level queuing, but a single lost packet still stalls every multiplexed stream because TCP itself delivers bytes strictly in order — this is TCP head-of-line blocking.
  - HTTP/3 replaces TCP with QUIC, a transport built on UDP that tracks each stream's data independently, so a lost packet only stalls the one stream it belongs to, not every call sharing the connection.
  - Keep-alive reuses an already-established TCP connection (and its TLS session) across multiple requests instead of renegotiating one per request, which matters enormously at scale because connection and TLS handshakes are among the most expensive fixed costs in a request's lifecycle.
check_questions:
  - q: Why would a service-to-service RPC call almost always default to TCP instead of UDP?
    a: TCP guarantees ordered, reliable delivery and handles retransmission and congestion control automatically, which matches what most RPC calls need — a request and response that must arrive complete and in order — whereas UDP offers no such guarantee and would push that reliability work back onto the application.
  - q: What is TCP head-of-line blocking, and why does HTTP/2 still suffer from it even though it multiplexes requests?
    a: TCP delivers bytes to the application strictly in the order they were sent, so if one packet is lost, every byte sent after it — even bytes belonging to a completely different HTTP/2 stream on the same connection — has to wait for that lost packet to be retransmitted and arrive before any of it can be delivered.
  - q: Why does HTTP/3 avoid the head-of-line blocking problem that HTTP/2 has?
    a: HTTP/3 runs over QUIC, which is built on UDP and tracks the data for each stream independently rather than as one single ordered byte sequence, so a packet loss affecting one stream only stalls that stream and lets the others keep delivering data normally.
related_formulas: []
---

## The transport layer is a design decision, not a default

This roadmap's computer-science track already covers what TCP and UDP are and how the OSI/TCP-IP stack fits together. What matters for system design is a different question: given that you're building a service that talks to other services, which transport protocol — and which HTTP version riding on top of it — actually fits the traffic pattern you have? These choices aren't free; they trade reliability, latency, and complexity against each other in ways that show up directly in tail latency and throughput once a system is handling real load.

## TCP vs UDP for service-to-service calls

Most RPC-style communication between services defaults to TCP, and for good reason: a service calling another service almost always needs the full response, in the right order, with corruption and loss handled transparently. TCP provides exactly that — ordered delivery, automatic retransmission of lost segments, and congestion control that backs off when the network is struggling — so the application code never has to think about any of it. The cost is a three-way handshake before any data flows, and built-in mechanisms (like slow-start) that can add latency on a fresh or underused connection.

UDP drops all of those guarantees: packets can arrive out of order, duplicated, or not at all, and the application gets no help detecting or fixing any of it. That sounds strictly worse, but it's the right tradeoff when an application doesn't need every packet and would rather drop stale data than wait for a retransmission that's no longer useful. A live video call losing one frame of audio is better off skipping it and moving on than freezing the whole stream to wait for a retransmit; a metrics-collection pipeline sending a datapoint every second can simply miss one if it is dropped. Service-to-service systems that build their own protocols on UDP (QUIC being the most consequential modern example — more below) do so because they want to add *selective* reliability without inheriting TCP's strict in-order delivery for everything.

## HTTP/1.1, HTTP/2, and HTTP/3: the multiplexing story

Assuming TCP is the transport, the next layer up — the HTTP version — determines how efficiently a service can issue many requests over a connection, which matters a great deal for RPC-heavy systems making dozens of calls to assemble one response.

**HTTP/1.1** allows only one request to be outstanding on a connection at a time (pipelining exists in the spec but is rarely used in practice because of how it interacts with head-of-line blocking at the HTTP layer). A client that wants concurrency has to open multiple TCP connections and spread requests across them — which works, but each connection has its own handshake and congestion-control ramp-up, and browsers and many HTTP clients cap how many connections they'll open to one host.

**HTTP/2** fixes the HTTP-layer part of this by multiplexing: many logical request/response streams share a single TCP connection, interleaved at the frame level, so a client can have dozens of requests in flight on one connection without needing more TCP handshakes. This is a real win for RPC-heavy systems that make many small calls — fewer connections to manage, better use of each connection's established congestion window. But HTTP/2 still rides on TCP, and TCP delivers bytes strictly in order. If one packet carrying data for stream 7 is lost, TCP won't deliver *any* subsequent bytes — including ones belonging to streams 1 through 6 that arrived just fine — until the lost packet is retransmitted and arrives. This is **TCP head-of-line blocking**: multiplexing solved the application-level queuing problem but inherited a transport-level one.

**HTTP/3** addresses this by changing the transport underneath HTTP entirely, running over **QUIC** instead of TCP. QUIC is built on UDP but implements its own reliability per stream rather than as one global ordered byte sequence — so a lost packet belonging to stream 7 stalls stream 7 alone, and streams 1 through 6 continue delivering their data to the application without waiting. QUIC also folds in what used to be a separate TLS handshake, typically needing fewer round trips to establish a secure connection than TCP+TLS. For an RPC-heavy system making many concurrent calls over a lossy or high-latency network (mobile clients, cross-region service calls), this removes a real source of tail-latency spikes that HTTP/2 can't avoid.

## Keep-alive: don't pay the handshake twice

Independent of which HTTP version is in use, **connection keep-alive** — reusing an already-open TCP connection (and, for HTTPS, its already-negotiated TLS session) across multiple requests instead of tearing it down and reconnecting for each one — is one of the highest-impact optimizations available at scale. Establishing a TCP connection costs at least one round trip; adding TLS on top costs one or two more; and a fresh TCP connection starts with a small congestion window that takes time to ramp up to full throughput. A service making hundreds of calls per second to a downstream dependency and reconnecting every time would spend more time on handshakes than on actual work. Keep-alive amortizes that fixed cost across many requests, which is why virtually every production HTTP client and server defaults to it, and why connection pooling (reusing a set of already-established connections rather than opening one per request) is a standard pattern for any service that talks to another service frequently.

## Putting it together

None of this is abstract trivia: a service mesh deciding whether to run gRPC over HTTP/2 or experiment with HTTP/3, a client library deciding how many connections to keep warm in a pool, or a team debugging a tail-latency spike that correlates with packet loss on a specific network path are all running into exactly these tradeoffs. Choosing TCP versus UDP, and HTTP/1.1 versus HTTP/2 versus HTTP/3, is choosing how a system behaves under the conditions that matter most — load and imperfect networks — not just how it behaves in a clean benchmark.
