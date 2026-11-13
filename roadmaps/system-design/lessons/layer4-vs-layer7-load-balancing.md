---
title: "Layer 4 vs. Layer 7 Load Balancing"
description: What a transport-layer load balancer can see and do versus an application-layer one — and the real cost of the difference.
roadmap: system-design
node: layer4-vs-layer7-load-balancing
group: load-balancing-and-service-discovery
video:
  provider: youtube
  id: "aKMLgFVxZYk"
  start: 0
  end: null
creator:
  name: Hussein Nasser
  channel_url: https://www.youtube.com/@hnasr
duration_seconds: 2252
level: intermediate
prerequisites:
  - load-balancing-algorithms
takeaways:
  - A Layer 4 load balancer only sees IP addresses and TCP/UDP ports, and makes its one routing decision for an entire connection's lifetime without ever reading the application payload.
  - Layer 4 balancing is fast and protocol-agnostic because it never reassembles the byte stream into requests, but that same blindness means it cannot route by URL path, header, or cookie.
  - A Layer 7 balancer terminates the client's connection, reassembles the actual HTTP request, and holds a separate connection to the backend, which is what lets it route on content and terminate TLS.
  - TLS termination specifically requires Layer 7, because decrypting HTTPS means understanding the handshake above the transport layer that Layer 4 never looks at.
  - A long-lived connection, such as a kept-alive HTTP/1.1 connection or a WebSocket, can cause load imbalance at Layer 4 because the balancer's one decision for that connection sticks for its entire lifetime.
  - Most production systems stack both layers, using a fast Layer 4 tier for coarse connection distribution and a Layer 7 tier behind it for content-aware routing.
check_questions:
  - q: Why can't a Layer 4 load balancer route requests differently based on URL path?
    a: It never reassembles the TCP byte stream into an actual HTTP request, so it has no access to the URL, headers, or any other application-layer content — it only ever sees IP addresses and ports.
  - q: Why does a single long-lived HTTP/1.1 keep-alive connection create a load-imbalance risk specifically at Layer 4?
    a: An L4 balancer makes its routing decision once, for the whole connection's lifetime, so every request sent over that one kept-alive connection is stuck going to the same backend it originally picked, even if that backend later becomes the most loaded one in the pool.
  - q: Why must TLS termination happen at Layer 7 rather than Layer 4?
    a: Terminating TLS means completing the HTTPS handshake and decrypting the traffic, which requires understanding protocol information above the transport layer, and a Layer 4 balancer by definition never inspects anything past IP addresses and ports.
related_formulas: []
---

## What an L4 balancer actually sees

A Layer 4 load balancer operates at the transport layer: it sees IP
addresses, TCP or UDP ports, and the raw stream of packets, and nothing
above that. It makes its routing decision once, early — typically by
inspecting the first packet of a new TCP connection — and from then on it
just forwards packets for that connection to the chosen backend, usually by
rewriting IP/port headers (NAT) or, in high-performance setups, by having
the backend respond directly to the client (Direct Server Return) so the
balancer never has to see the response traffic at all.

Because an L4 balancer never terminates the TCP connection — it never
reassembles the byte stream into HTTP requests, never looks at a URL path, a
header, or a cookie — it is fast. It can run in the kernel or on dedicated
hardware, routing packets at line rate with latency overhead in the
microseconds. It is also protocol-agnostic: it balances TCP or UDP traffic
for anything, from HTTP to a custom binary protocol to a database
connection, because it never needed to understand the payload in the first
place.

The cost of this speed is blindness. An L4 balancer cannot route
`/api/v1/search` differently from `/api/v1/checkout`. It cannot see that a
request is carrying an expired auth cookie. It cannot terminate TLS and hand
plaintext to the backend, because doing that requires understanding the
HTTPS handshake, which lives above layer 4. All it can do is spread
*connections* across backends — which is also why sticky, long-lived
connections (like a WebSocket, or an HTTP/1.1 keep-alive connection reused
for hundreds of requests) can cause imbalance at L4: the balancer made one
decision for the whole connection's lifetime, and if that connection happens
to live for an hour and carry ten thousand requests, that's ten thousand
requests stuck on one backend.

## What L7 buys you

A Layer 7 load balancer terminates the client's connection itself, reads and
reassembles the actual application-layer request — almost always HTTP(S) —
and only then decides where to send it. Practically, this means it holds two
separate TCP connections: one to the client, one to whichever backend it
picks, and it actively proxies data between them rather than just forwarding
packets.

Because it's reading the request, it can act on everything in it: route
`/static/*` to a CDN-backed pool and `/api/*` to application servers, route
based on a `Host` header to support multiple domains behind one balancer,
inspect a cookie to keep a specific user pinned to the server holding their
session, or reject a malformed request before it ever reaches a backend.
Most significantly for production systems, an L7 balancer is almost always
where TLS termination happens: the balancer holds the certificate, decrypts
the client's HTTPS traffic, and can then either forward plaintext to
backends on a trusted internal network or re-encrypt for a second TLS hop —
either way, backends stop needing to each manage their own certificates.

The next lesson on reverse proxies and API gateways goes further into what
gets layered on top of this (caching, compression, auth); the point to take
from this lesson is narrower: once you're willing to terminate the
connection and read the payload, URL- and header-based routing and TLS
termination become possible, and none of them are possible at L4.

## The cost of looking

That visibility is not free. Reassembling TCP segments into HTTP messages,
parsing headers, and running a second TLS handshake to the backend (if
re-encrypting) all cost CPU — a few hundred microseconds to low milliseconds
per request on modern hardware, which is real but usually negligible next to
backend processing time. The bigger operational cost is that an L7 balancer
is now a full HTTP-aware proxy: it needs to handle malformed requests, slow
clients holding connections open (a mitigation surface for denial-of-service
attacks that L4 doesn't have to think about), and its own connection pool to
backends, with its own timeouts and retry logic.

## Choosing between them

In practice, most production edges run both, stacked: an L4 balancer (often
a managed one like a cloud network load balancer, or a kernel-level one like
Linux's IPVS) sits at the very edge handling the raw flood of TCP
connections and doing coarse-grained distribution across a layer of L7
balancers or reverse proxies, which then do the content-aware routing. The
decision isn't really "L4 or L7" for a whole system — it's "which layer
does this specific decision belong at." Anything that only needs "spread
connections across healthy hosts" belongs at L4, where it's cheapest.
Anything that needs to read the request — path-based routing, TLS
termination, cookie-based session affinity — has to happen at L7, because
that information simply doesn't exist yet at the point an L4 balancer makes
its decision.
