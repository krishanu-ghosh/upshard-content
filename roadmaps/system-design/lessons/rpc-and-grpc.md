---
title: "RPC and gRPC"
description: What RPC buys you over raw sockets, how gRPC and Protocol Buffers fit together, and when synchronous RPC is the wrong choice.
roadmap: system-design
node: rpc-and-grpc
group: networking-apis-and-communication
video:
  provider: youtube
  id: "DVTitCGwh6g"
  start: 0
  end: null
creator:
  name: SystemDR - Scalable System Design
  channel_url: https://www.youtube.com/@SystemDR
duration_seconds: 545
level: intermediate
prerequisites:
  - choosing-transport-protocols-for-services
takeaways:
  - RPC lets a developer call a function on a remote service as if it were a local function call, hiding the serialization, network transport, and deserialization behind a generated stub — raw sockets give you none of that, leaving you to hand-roll a wire format and framing logic yourself.
  - gRPC is a specific, opinionated RPC framework that pairs Protocol Buffers (a compact, strongly-typed binary format) with HTTP/2, and uses a schema-first contract — you define the service and message shapes in a .proto file before you write any implementation code.
  - Codegen is central to gRPC's value — the same .proto file generates client and server code in multiple languages, so a Java service and a Go service can call each other with compile-time type checking on both ends instead of trusting a hand-maintained API document.
  - Beyond simple request/response, gRPC supports streaming RPCs — a client or server (or both) can send a sequence of messages over one call instead of a single request and single reply, which raw REST-over-HTTP/1.1 can't do cleanly.
  - Synchronous RPC blocks the caller until the callee responds, which is simple to reason about but couples the two services' availability together; an asynchronous alternative like a message queue decouples that availability at the cost of giving up an immediate return value, and a system with a slow, unreliable, or optional downstream step is usually better served by that asynchronous path instead of RPC.
check_questions:
  - q: What does an RPC framework handle for you that raw TCP sockets don't?
    a: An RPC framework handles serializing arguments into a wire format, sending and receiving them over the network, deserializing the result, and presenting the whole thing to the calling code as an ordinary function call — with raw sockets, you'd have to design that wire format, frame messages so the receiver knows where one ends and the next begins, and write all the (de)serialization logic yourself.
  - q: Why does gRPC's schema-first approach with .proto files matter in practice?
    a: Because the service and message definitions live in one shared .proto file, client and server code in potentially different languages gets generated from the exact same contract, so a mismatch between what a client sends and what a server expects becomes a compile-time or codegen-time error instead of a runtime surprise discovered in production.
  - q: When is synchronous RPC the wrong choice for connecting two services?
    a: When the caller shouldn't be blocked waiting on the callee — for example if the downstream work is slow, unreliable, or not required to happen before the caller can finish — in which case an asynchronous approach like a message queue or event stream decouples the two services' availability and lets the caller proceed without waiting for a reply.
deeper:
  - label: "13 Introduction to gRPC and Protocol Buffers"
    url: "https://www.youtube.com/watch?v=pMgty_RYIOc"
related_formulas: []
---

## RPC: making a network call look like a function call

**Remote Procedure Call (RPC)** is an old idea with a simple goal: let a developer write `response = getUser(42)` and have that call actually travel across a network to another machine, without the caller having to think about sockets, byte layouts, or retries at the call site. The RPC framework's job is everything in between — taking the arguments, encoding them into some agreed-upon format, sending them over the network, waiting for a response, decoding it, and handing back a normal-looking return value (or raising an error if something went wrong).

Compare that to working with **raw sockets** directly. If you open a TCP socket to a remote service yourself, you get a stream of bytes and nothing else. You have to decide on a message format, figure out how the receiver knows where one message ends and the next begins (a framing problem — TCP gives you a byte stream, not discrete messages), write encoding and decoding logic for every type of request and response, and handle partial reads, reconnects, and timeouts by hand. None of that is wrong to do, but it's a lot of infrastructure to rebuild for every service that wants to talk to another service. RPC frameworks exist so you build that machinery once and reuse it.

## Where gRPC fits: an opinionated, modern RPC framework

**gRPC** is Google's open-source RPC framework, and it makes specific choices at each layer where plain "RPC" leaves room for options:

- **Wire format:** gRPC uses **Protocol Buffers** (protobuf) — a compact binary serialization format — rather than a text format like JSON. A protobuf-encoded message is smaller and faster to parse than the equivalent JSON, which matters when a service is making thousands of calls per second.
- **Transport:** gRPC runs over HTTP/2, inheriting its multiplexing (many calls share one connection without head-of-line blocking at the HTTP layer) and native support for long-lived, bidirectional streams.
- **Contract definition:** Before writing any server or client code, you write a `.proto` file describing the service's methods and the shape of its messages:

```protobuf
service UserService {
  rpc GetUser (GetUserRequest) returns (GetUserResponse);
}

message GetUserRequest {
  int32 user_id = 1;
}

message GetUserResponse {
  int32 user_id = 1;
  string name = 2;
  string email = 3;
}
```

This is what makes gRPC **schema-first**: the contract is written down and agreed on *before* implementation, not inferred from whatever a REST endpoint happens to return.

## Codegen: one contract, many languages, compile-time safety

The `.proto` file isn't just documentation — it's fed into the `protoc` compiler (with a gRPC plugin) to generate real client and server code. A team running a Go backend and a team running a Java backend can both generate code from the identical `.proto` file, and each gets native-feeling client stubs and server interfaces in their own language. If one side's message shape changes incompatibly, that mismatch tends to surface at build or codegen time rather than as a runtime parsing failure discovered after a bad deploy — a meaningfully stronger guarantee than a REST API where the only contract enforcement is whatever the two teams write down in a document and hope stays accurate.

gRPC also supports **streaming** beyond simple request/response: a client can stream a sequence of messages to the server (client streaming), a server can stream a sequence of responses back (server streaming), or both sides can stream simultaneously over one long-lived call (bidirectional streaming). A service sending live sensor readings, or a server pushing incremental results as they become available, can use this directly — something that's awkward to express cleanly with REST over HTTP/1.1's one-request-one-response model.

## Synchronous RPC vs. the asynchronous alternative

gRPC calls, like most RPC, are typically **synchronous**: the caller sends a request and blocks until a response (or an error, or a timeout) comes back. This is simple to reason about — the calling code's control flow matches what actually happens on the wire — but it has a real cost: the caller's availability and latency are now coupled to the callee's. If the downstream service is slow or down, the caller is stuck waiting (or has to implement its own timeout and retry logic on top).

A separate lesson in this roadmap covers message queues and pub/sub in depth, so the comparison here is deliberately brief: an **asynchronous** alternative — publishing a message to a queue or event stream instead of calling a service directly — decouples the two services. The publisher doesn't wait for the consumer to process anything, and the consumer doesn't even need to be running at the moment the message is published. The tradeoff is that you give up an immediate return value; the publisher can't get "the answer" back synchronously, only a confirmation that the message was accepted.

## When RPC is the wrong tool

Reach for RPC (gRPC or otherwise) when you need a request handled right now and need the result back in order to proceed — a payment-authorization check before completing a checkout, for instance. Reach for the asynchronous alternative instead when the work doesn't need to block the caller, when the downstream step is allowed to fail and be retried independently, when you want to fan a single event out to multiple unrelated consumers, or when the caller and callee shouldn't need to be online at the same instant for the system to make progress. Forcing RPC onto that second category of problem tends to produce a system that's more fragile than it needs to be — every synchronous call is a dependency that can turn one service's bad day into every upstream caller's bad day too.
