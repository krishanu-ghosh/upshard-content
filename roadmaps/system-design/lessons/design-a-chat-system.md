---
title: "Case Study: Design a Chat System"
description: A worked design for a WhatsApp/Messenger-style chat system, covering real-time delivery, message ordering, and presence at scale.
roadmap: system-design
node: design-a-chat-system
group: case-studies
video:
  provider: youtube
  id: "vvhC64hQZMk"
  start: 0
  end: null
creator:
  name: Gaurav Sen
  channel_url: https://www.youtube.com/@gkcs
duration_seconds: 1514
level: advanced
prerequisites:
  - message-queues-and-async-communication
  - service-discovery
takeaways:
  - A chat system's hardest problem isn't storing messages, it's knowing which server currently holds an open connection to a given user so a message can actually reach them.
  - WebSockets are the standard choice for real-time chat because they give a persistent, bidirectional connection, avoiding the repeated request overhead of long polling.
  - A per-sender, per-conversation sequence number gives messages a stable order for display even when they arrive at the server slightly out of order over the network.
  - At-least-once delivery plus client-side deduplication (via a message ID) is the practical way most chat systems guarantee a message is never silently lost, accepting that a rare duplicate is easier to filter than a missing message is to notice.
  - Presence ("online now") is usually approximate by design — a short heartbeat interval and a few seconds of staleness tolerance keep the presence system cheap, in exchange for it occasionally being a few seconds behind reality.
  - A connection-routing layer (tracking which gateway server holds which user's live connection) is what lets a stateless-looking chat backend actually deliver messages in real time.
check_questions:
  - q: Why do most real-time chat systems use WebSockets instead of HTTP long polling?
    a: Long polling requires the client to repeatedly open a new HTTP request that the server holds open until there's a message (or a timeout), which adds connection-setup overhead on every cycle, while a WebSocket opens one persistent, bidirectional connection that the server can push messages through instantly without the client re-requesting anything.
  - q: Why can't a chat server rely purely on the order messages arrive at the server to display them correctly to users?
    a: Network delays and retries can cause messages to arrive at the server slightly out of the order they were actually sent, so the server has to use a sender-assigned per-conversation sequence number (or timestamp plus tiebreaker) to reconstruct the intended order for display, rather than trusting arrival order.
  - q: Why does at-least-once delivery require client-side deduplication to be a complete guarantee?
    a: At-least-once delivery means the system will retry sending a message until it's acknowledged, which can occasionally deliver the same message twice if an acknowledgment is lost in transit even though the message itself arrived, so the client has to recognize and discard a duplicate by its message ID to avoid showing the same message twice to the user.
deeper:
  - label: "FAANG System Design Interview: Design A Chat System (ByteByteGo)"
    url: "https://www.youtube.com/watch?v=okrR1KXNLtA"
related_formulas: []
---

## Requirements

"Design a chat system" means building something like WhatsApp, Messenger, or Slack: one-to-one and group messaging, delivered close to real time, with message history and presence.

Functional requirements:
- Send and receive messages in one-to-one and group conversations.
- Persist message history so a user can scroll back or reconnect and catch up.
- Show delivery state (sent, delivered, read) and presence (online, last seen).
- Support users who are offline when a message is sent — the message must wait for them.

Non-functional requirements:
- **Low latency delivery**: a message should reach an online recipient in well under a second.
- **Message ordering**: messages within a conversation need a stable, consistent order for every participant.
- **Durability**: a message, once accepted by the server, must not be lost even if the recipient is offline for days.
- **Massive concurrent connections**: unlike a typical stateless web service, this system needs millions of simultaneously *open* connections, not just requests-per-second.

## Capacity estimate

Suppose the system has 50 million daily active users, each sending an average of 40 messages/day, with a peak-to-average ratio of 5x.

- Messages/day: 50M × 40 = 2 billion messages/day ≈ 23,000 messages/second average, ~115,000/second at peak.
- Concurrent open connections: if 20% of daily actives are online simultaneously at peak, that's 10 million concurrent WebSocket connections to hold open — this, not raw message throughput, is usually the defining constraint, since each connection consumes server-side memory and file descriptors even while idle.
- If a single connection-handling server can hold roughly 50,000-100,000 concurrent connections (a realistic figure for a tuned WebSocket gateway), 10 million connections need on the order of 100-200 such gateway servers just for connection handling, separate from the servers doing message routing and storage.
- Storage: at ~200 bytes/message (including metadata), 2 billion messages/day × 200 bytes ≈ 400 GB/day, or roughly 150 TB/year — large but entirely tractable with a sharded storage layer.

## High-level design

Each client holds a persistent WebSocket connection to one of many stateful **connection gateway** servers. A separate, lightweight **presence/connection-routing service** (often backed by a fast key-value store) tracks which gateway server currently holds each user's live connection — this mapping is the crucial piece of state that makes real-time delivery possible at all. When user A sends a message to user B, it hits A's gateway, gets written durably to a message store (sharded by conversation ID) and pushed onto a message queue for fan-out, then the routing service is consulted to find B's gateway (if B is online) so the message can be pushed down B's open connection immediately. If B is offline, the message simply waits in durable storage; when B reconnects, their client syncs any messages that arrived while they were away. Group chats fan a message out to every member's respective gateway connection, using the same routing lookup per recipient (see [message queues and async communication](/roadmaps/system-design/message-queues-and-async-communication) for the queueing layer that decouples "message accepted" from "message delivered").

## Key decisions

**1. Real-time delivery mechanism.** Long polling (client repeatedly asks "anything new?" and the server holds the request open until there is) works but adds connection-setup overhead on every cycle and scales poorly to very high message rates. Server-sent events give efficient one-way server-to-client push but no return channel for the client to send messages the same way. WebSockets give a single persistent, bidirectional connection — the client sends and receives over the same socket — which is why essentially every production chat system settles on them for the live-messaging path, reserving plain HTTP for less latency-sensitive operations like fetching history.

**2. Message ordering and delivery guarantees.** Network delays mean messages can arrive at the server (or at other participants) slightly out of the order they were sent, so each message carries a sender-assigned, per-conversation sequence number (or a timestamp with a tiebreaker) that clients use to render the correct order regardless of arrival order. On the delivery-guarantee side, exactly-once delivery is expensive to guarantee end-to-end, so most systems accept at-least-once (retry until acknowledged) paired with a client-side dedup step keyed on message ID — a rare duplicate that gets silently dropped by the client is a far better failure mode than a message that silently vanishes (see [idempotent APIs and exactly-once semantics](/roadmaps/system-design/idempotent-apis-and-exactly-once-semantics)).

**3. Presence and online status at scale.** Tracking "is this user online right now" precisely, for tens of millions of users, in real time, would require constant chatter between every interested party and every user's connection state. Real systems relax this deliberately: clients send periodic heartbeats (every 10-30 seconds), and a user is considered online until a heartbeat is missed past some grace period — meaning presence is always *slightly* stale by design, in exchange for being cheap to compute and propagate. Pushing presence changes to a user's contacts, rather than having every client poll for it, further cuts load, but it means "online" badges across the whole system are a best-effort approximation, not a real-time guarantee — a tradeoff users generally don't notice and systems can't economically avoid at this scale.

## Tradeoff and failure mode worth calling out

The failure mode most worth planning for is a **gateway server crashing while holding millions of open connections**. Every client attached to that gateway loses its connection simultaneously and reconnects at roughly the same moment — a classic reconnection stampede that can overwhelm the load balancer and the routing service right as the system is already short one gateway's worth of capacity. The routing service's mapping for every affected user is now stale until each client reconnects and re-registers, meaning any message sent to one of those users in that window needs to queue durably rather than assume an active connection exists — which is precisely why durable storage and fan-out happen *before* the routing lookup in the design above, not after. Mitigating the stampede itself usually means client-side reconnect backoff with jitter, so ten million clients don't all retry in the same half-second.
