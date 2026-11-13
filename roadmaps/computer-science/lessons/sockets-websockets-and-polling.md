---
title: "Sockets, WebSockets, and Polling"
description: What a network socket is, and the three ways browsers get near-real-time updates from a server — short polling, long polling, and WebSockets.
roadmap: computer-science
node: sockets-websockets-and-polling
group: networking-fundamentals
video:
  provider: youtube
  id: "WS352jTTkPU"
  start: 0
  end: null
creator:
  name: Piyush Garg
  channel_url: https://www.youtube.com/@piyushgargdev
duration_seconds: 1643
level: intermediate
prerequisites:
  - tls-and-https
takeaways:
  - "A socket is an endpoint for two-way communication between two programs over a network, identified by an IP address and a port; TCP and UDP both hand your application a socket to read from and write to."
  - "Plain HTTP is request-response: the client always speaks first, and the server can never push data to the client on its own initiative, which is a problem for anything that needs near-real-time updates (chat, live scores, notifications)."
  - "Short polling works around this by having the client repeatedly ask \"anything new?\" on a timer, which is simple but wastes requests when nothing has changed and adds latency up to one poll interval when something has."
  - "Long polling improves on that by having the server hold the request open until there's actually something to report (or a timeout passes), cutting wasted round trips at the cost of holding more open connections on the server."
  - "WebSockets solve the underlying problem directly: after an HTTP handshake upgrades the connection, both sides keep a single persistent, full-duplex socket open and can send messages to each other at any time, with no polling at all."
check_questions:
  - q: Why can't a plain HTTP server just push a new chat message to the browser the moment it arrives?
    a: HTTP is strictly request-response — the server can only ever reply to a request the client already sent, so without the client asking again, the server has no channel to use to send something unprompted.
  - q: What's the main cost of long polling compared to short polling, given that it sends fewer wasted requests?
    a: Long polling holds each request open on the server until there's something to report, which means the server has to keep many more connections open and waiting at once than it would with short polling's quick request-reply cycles, costing server resources instead of network requests.
  - q: What has to happen before a WebSocket connection can start sending full-duplex messages?
    a: The client sends a normal HTTP request asking to "upgrade" the connection, and if the server agrees, that same TCP connection is repurposed into a persistent WebSocket connection instead of being closed after the response, as an ordinary HTTP connection would be.
  - q: If WebSockets are more efficient than polling, why does polling still get used at all?
    a: WebSockets require the server (and often the network path — some proxies and firewalls handle them poorly) to support a persistent connection per client, which is more infrastructure than some systems want to maintain, so many applications use polling for its simplicity when true real-time delivery isn't critical.
deeper:
  - label: "Short Polling vs Long Polling vs WebSockets - System Design"
    url: "https://www.youtube.com/watch?v=ZBM28ZPlin8"
related_formulas: []
---


## Sockets: the thing underneath every network connection

Every network connection your browser or any other program makes, whether it knows it or not, goes through a **socket**: an operating-system-provided endpoint for sending and receiving data, identified by a combination of an IP address and a port number. When your browser opens a TCP connection to a web server, the OS hands it a socket; reading from that socket gives you the bytes the server sent, and writing to it sends bytes back. HTTP, TLS, and everything built on top of them are ultimately just protocols for what bytes get written to and read from a socket — the socket itself is protocol-agnostic plumbing.

## The limit built into plain HTTP

HTTP's request-response model has a specific shape: the client sends a request, the server sends back exactly one response, and the exchange is over. The server has no way to initiate contact — it can only ever respond to something the client already asked. That's a fine model for loading a page, but it breaks down the moment you want the browser to learn about something the *server* knows happened, without the user manually refreshing: a new chat message arriving, a price changing, a background job finishing. The server has no channel to push that information through.

## Short polling: ask again, and again

The simplest fix is **short polling**: the client just asks repeatedly, on a fixed interval — "anything new?" every two seconds, say. If nothing's changed, the server replies immediately with "no." If something has, the server replies with the update, and the client processes it and goes back to asking.

This is trivial to implement and works with ordinary HTTP requests, no special infrastructure needed. The cost is twofold: most requests are wasted (the overwhelming majority of the time, the answer is "nothing new," but the client still pays the overhead of sending a request and the server pays the cost of answering it), and updates are delayed by up to one full poll interval even when the server has the information ready to send the instant it happens.

## Long polling: hold the line open until there's something to say

**Long polling** keeps the same request-response shape but changes what the server does with the request: instead of answering "no" immediately when there's nothing new, the server holds the connection open — doesn't respond yet — and waits until either something actually happens or a timeout is reached. The moment there's an update, the server responds with it right away; the client immediately opens a fresh long-poll request to keep listening for the next one.

This eliminates almost all of the wasted "no" responses and gets updates to the client close to the moment they occur, at the cost of a different resource: the server now has to keep many requests open and waiting simultaneously, one per connected client, which uses up server-side connection slots and memory that short polling's quick, in-and-out requests didn't need.

## WebSockets: stop simulating two-way communication and just have it

Both polling approaches are, fundamentally, ways of faking two-way communication on top of a protocol that was only ever designed for one-way requests. **WebSockets** remove the need for the workaround by actually providing a two-way (full-duplex) channel. The connection starts as an ordinary HTTP request that asks to "upgrade" — a special header tells the server the client wants to switch protocols. If the server agrees, the same underlying TCP connection that carried that HTTP request is repurposed: it stays open indefinitely, and from that point on, either side — client or server — can send a message to the other at any time, with no request/response pairing required at all.

That's a categorically different model from polling: the server doesn't wait to be asked, and the client doesn't have to re-ask on a timer. A chat message, a live score update, or a stock price tick can be pushed to every connected client the instant it happens, over a connection that's already open and waiting.

## Picking between them

The tradeoff isn't purely "WebSockets are strictly better" — they require the server (and anything in between, like load balancers and proxies) to support holding open a persistent connection per client, which is real infrastructure to build and operate, and some older or more restrictive network setups handle long-lived connections poorly. For something that updates rarely and where a few seconds of staleness is fine, short polling's simplicity is often the right engineering tradeoff. For something that genuinely needs sub-second, bidirectional updates at scale — live collaborative editing, multiplayer games, trading dashboards — WebSockets are usually worth the added operational complexity. Long polling tends to sit in between: a pragmatic choice when you want low latency but don't want to take on full WebSocket infrastructure.
