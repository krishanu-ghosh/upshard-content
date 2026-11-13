---
title: "Streaming and Real-Time API Contracts"
description: Exposing real-time data as a documented API surface — choosing between SSE, WebSockets, and long-polling, and designing an event schema clients can build to.
roadmap: api-design
node: streaming-and-real-time-api-contracts
group: real-time-and-integration-patterns
video:
  provider: youtube
  id: "RWaX_qU1d0E"
  start: 0
  end: null
creator:
  name: Code and Stuff
  channel_url: https://www.youtube.com/@CodeAndStuff
duration_seconds: 587
level: intermediate
prerequisites: []
takeaways:
  - "Choosing a real-time transport for a public API is a product decision, not just a technical one: SSE fits one-way server-to-client feeds (notifications, live scores, status updates) that need to survive proxies and reconnect on their own; WebSockets fit genuinely bidirectional traffic (chat, collaborative editing) where the client needs to push data just as often as it receives it."
  - "A real-time event needs the same contract discipline as any REST response: a stable event type name, a schema version, and a payload shape documented well enough that a client written against today's events doesn't silently break when a field changes."
  - "Backpressure, from the API consumer's side, means deciding and documenting what happens when a subscriber can't keep up with the stream — buffer and risk unbounded memory, drop the oldest or newest events, or disconnect the slow client — because \"it'll be fine\" is not a contract."
  - "Versioning a real-time event stream is harder than versioning a REST endpoint because a client can be mid-connection when a breaking change ships; the safe pattern is additive-only changes to an existing event version, with a genuinely new event type (or a payload `version` field clients can branch on) for anything breaking, run in parallel until old subscribers drain."
  - "`Last-Event-ID` (SSE's native resume mechanism) and a client-supplied cursor or sequence number (for WebSocket or custom protocols) exist to answer the same question — \"what did I miss while disconnected?\" — and a real-time API that doesn't answer it is only reliable as long as the network is, which is never a safe bet for a public API."
  - "Long-polling is rarely the right permanent choice for a new public API today, but it stays relevant as the compatibility fallback for clients or intermediaries (corporate proxies, some mobile carriers) that mishandle long-lived connections — a real-time API aimed at a wide, uncontrolled client population sometimes has to offer it alongside SSE or WebSockets, not instead of them."
check_questions:
  - q: "Why is picking SSE vs. WebSockets for a public API a product decision and not purely a technical one?"
    a: "The choice depends on what the client population actually needs to do — sometimes-write-sometimes-read access like chat needs full duplex, while dashboards and notification feeds only ever need to receive — so the right answer tracks the use case and client diversity, not which protocol is newer or more capable in the abstract."
  - q: "What does it mean to version a real-time event stream safely, and why is it harder than versioning a REST endpoint?"
    a: "It means only ever adding fields or new event types to an existing stream, never changing or removing what's already there, because unlike a REST request a client can be mid-connection, having already parsed earlier events, when a change ships — there's no single request/response moment to attach a version negotiation to, so the stream itself has to stay backward compatible or the API has to run two versions in parallel."
  - q: "A client's WebSocket connection drops for 90 seconds during a network blip. What does a well-designed real-time API contract let that client do when it reconnects?"
    a: "Resume from where it left off — using SSE's built-in `Last-Event-ID` header or an equivalent client-supplied cursor/sequence number over WebSockets — so the server can replay (or let the client fetch) anything published during the gap, instead of leaving the client to either assume it missed nothing or re-fetch the entire resource state from scratch."
  - q: "What's the actual engineering decision behind \"backpressure\" when a real-time API's subscriber can't keep up?"
    a: "The server has to pick one of a small set of explicit behaviors — buffer events up to some bound and apply backpressure to the producer, drop events (oldest-first or newest-first) once a buffer fills, or disconnect the slow consumer outright — and document which one it does, because an unbounded buffer just turns a slow client into a server-side memory leak instead of solving the problem."
deeper:
  - label: "Event Versioning Guidelines"
    url: "https://www.youtube.com/watch?v=L8eH1XaRnPw"
related_formulas: []
---

## This is a contract problem, not a transport problem

`computer-science`'s `sockets-websockets-and-polling` already covers how short polling, long polling, and WebSockets actually work under the hood — the HTTP upgrade handshake, what a persistent socket is, why the server can't push over plain HTTP. This lesson assumes you have that mechanical picture and asks a different question: once you've decided your API needs to deliver real-time updates, what do you actually ship as a *public, documented API surface* that other teams or outside developers build integrations against? That's a product and contract design problem, and it's where most real-time APIs actually go wrong — not in the transport implementation, but in treating the stream as an implementation detail instead of a versioned interface.

## Picking a transport is about your client population, not your server

Three real options exist for exposing real-time data publicly: **Server-Sent Events**, **WebSockets**, and **long-polling** (sometimes alongside, not instead of, one of the first two). The decision is driven by what your clients actually need to do:

- **SSE** is the right default for anything that's fundamentally one-way: price ticks, build status, notification feeds, live dashboards. It runs over plain HTTP, so it passes through existing infrastructure (load balancers, corporate proxies, API gateways) far more reliably than a raw WebSocket upgrade does, and the browser's `EventSource` handles reconnection and resumption (via `Last-Event-ID`) for free — a surprisingly large chunk of "real-time API reliability" that you'd otherwise have to build yourself.
- **WebSockets** earn their operational cost — persistent full-duplex connections, your own reconnect and resume logic, proxy-compatibility headaches — only when clients genuinely need to send as much as they receive: chat, multiplayer state, collaborative editing. If your API consumers only ever read a stream and occasionally issue a normal REST call to act on it, you are very likely paying WebSocket complexity for an SSE use case.
- **Long-polling** is the pattern you offer when you can't control your client population and some meaningful slice of it — older proxies, restrictive corporate networks, SDKs you don't maintain — can't reliably hold open a streaming connection. A public API with a wide, heterogeneous integrator base (think: a payments platform's webhook-adjacent event feed, used by thousands of third-party developers on unknown infrastructure) sometimes needs it as a documented fallback mode, not a demo-day afterthought.

The wrong way to make this decision is "WebSockets are the most powerful, so we'll expose those." Powerful and appropriate aren't the same axis, and every bidirectional connection you don't need is server-side state you now have to scale, load-balance with sticky routing or a shared pub/sub backplane, and keep alive through deploys.

## Designing the event schema

A real-time event is a response just like any REST payload, and it deserves the same discipline `rest-resource-modeling-and-uri-design` asks of a resource body: a **stable, explicit event type** (`invoice.payment_failed`, not a generic `{type: "update"}` blob the client has to sniff), a **unique event id** for dedup and logging, and a **documented payload shape** per event type. Resist the temptation to ship "whatever the internal domain object looks like this week" as the event body — that's the same mistake as leaking an ORM model through a REST endpoint, just replayed on every message instead of once per request. The stream is public API surface; it needs its own DTO, decoupled from whatever representation your service uses internally, for exactly the reason `structured-error-handling-and-rfc-7807` argues for a stable error shape: so an internal refactor doesn't silently break every client parsing your messages.

## Versioning a stream you can't pause

REST versioning (`api-versioning-strategies`, elsewhere in this roadmap) gets to lean on the fact that every request is a fresh negotiation — a client can send `Accept: application/vnd.api+json;version=2` and get routed accordingly. A long-lived stream doesn't get that luxury: a client can be mid-connection, already holding state built from earlier events, when you want to ship a breaking change. The safe discipline is **additive-only evolution within an event version** — new optional fields are fine, because clients ignore keys they don't recognize — and a **genuinely new event type or an explicit `schema_version` field** for anything that changes meaning or removes a field. When you do need a breaking change, run both versions of the stream in parallel (a new event type alongside the old, or a versioned endpoint/channel) until you can confirm, from your own delivery logs, that nothing is still subscribed to the old shape. There is no clean single moment to "cut over" a stream the way there is for a REST endpoint; plan for the overlap period as a first-class part of the rollout, not a scramble.

## Backpressure is the API consumer's problem too

Backpressure usually gets discussed from the server's side — how do you protect yourself from a producer that's faster than your consumers. From the *consumer's* side, as a documented API contract, backpressure means answering one specific question in your docs: **what happens when my client can't keep up?** A slow consumer on a bounded internal queue is one thing; a slow consumer on a public SSE or WebSocket stream, where you don't control the client's processing speed at all, is a decision you have to make explicit. The honest options are: buffer up to a bound and disconnect once it's exceeded, drop events (and tell the client, via a sequence gap or an explicit "you missed N events" message, that it happened), or apply true flow control if your transport supports it. "We'll just buffer everything" is not a real answer for a public API — it's an unbounded memory commitment to every slow client that ever connects, and it's exactly the kind of thing that looks fine in a demo and pages someone at 3 a.m. once real traffic arrives. Document the behavior the same way you'd document a rate limit, because to the client building against your API, it's the same kind of constraint: a hard edge they need to design around, not a surprise they discover in production.

The next lesson, `webhooks-vs-polling`, covers the other half of "real-time-ish" API design — delivery you push to a client's own server rather than a connection they hold open to yours, and the very different guarantees (at-least-once, no ordering) that come with it.
