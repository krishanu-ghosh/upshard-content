---
title: "The Actor Model: Concurrency Without Shared Memory"
description: How actors, mailboxes, and message passing let concurrent code stay correct by making shared mutable state structurally impossible.
roadmap: software-architect
node: the-actor-model
group: paradigms
video:
  provider: youtube
  id: "ELwEdb_pD0k"
  start: 0
  end: null
creator:
  name: Finematics
  channel_url: https://www.youtube.com/@Finematics
duration_seconds: 272
level: intermediate
prerequisites:
  - reactive-programming
takeaways:
  - "An actor is an independent unit of computation that holds its own private state, which no other actor can read or write directly — the only way to interact with an actor is to send it a message."
  - "Every actor has a mailbox, a queue of incoming messages it processes one at a time, in order, which means an actor's own state is never touched by two operations simultaneously, even though many actors run concurrently."
  - "This sidesteps the entire class of problems computer-science's `synchronization-locks-and-semaphores` and `race-conditions-and-thread-safety` exist to manage, by removing shared mutable memory from the model entirely rather than carefully guarding access to it."
  - "An actor can create new actors, send messages to other actors it knows about, and decide how to handle the next message — which is often used to implement supervision, where a parent actor restarts a child that crashes instead of letting the failure propagate."
  - "The trade-off is that actor systems generally do not guarantee message delivery or ordering across actors the way a function call guarantees its result, so the pattern pushes failure handling (timeouts, retries, supervision) to the front of the design rather than letting it be an afterthought."
check_questions:
  - q: Why can't two messages to the same actor ever cause a race condition on that actor's own state?
    a: Because each actor processes messages from its mailbox strictly one at a time, in order — even if a thousand messages arrive concurrently from different senders, they queue up and the actor handles them sequentially, so there is never a moment where two operations are reading or writing that actor's state at once.
  - q: How does the actor model's approach to concurrency differ fundamentally from using locks and semaphores on shared memory?
    a: Locks and semaphores let multiple threads share the same memory and coordinate access to it through careful discipline, which is correct only if every piece of code that touches that memory remembers to take the lock; the actor model removes shared mutable memory from the picture entirely, so there's nothing to forget to lock — isolation is structural, not a matter of discipline.
  - q: What does "let it crash" mean in an actor system, and why is supervision what makes that survivable?
    a: The phrase "let it crash" means an actor that hits an error it can't handle is simply allowed to stop rather than trying to defensively catch and recover from every possible failure inline; a supervising parent actor is watching for that crash and restarts the child actor in a known-good state, so the system recovers automatically instead of needing every actor to anticipate every failure mode itself.
related_formulas: []
---

## The problem: shared mutable state under concurrency

Every approach to concurrent programming has to answer one question: what happens when two things run at the same time and both want to touch the same piece of state? The traditional answer — threads sharing memory, coordinated by locks and semaphores — is covered in depth in `computer-science`'s `synchronization-locks-and-semaphores` and `race-conditions-and-thread-safety`, and it works, but it demands constant discipline: every single piece of code that touches shared state has to remember to take the right lock, in the right order, every time, or the program has a race condition that might not show up until it's running in production under load. The actor model, dating to Carl Hewitt's 1973 formulation and popularized through Erlang and later Akka, takes a structurally different approach: instead of disciplining access to shared memory, it removes shared mutable memory from the model entirely.

## What an actor actually is

An actor is an independent unit of computation with three capabilities and one hard rule. It can: hold private state that no other actor can see or touch directly; send messages to other actors it has a reference to; and create new actors. The hard rule is that the *only* way to interact with an actor from the outside is to send it a message — there is no shared variable, no direct method call into its internals, nothing to synchronize because there is nothing shared to begin with. Each actor has a **mailbox**, a queue that holds incoming messages until the actor is ready to process them, and the actor handles messages from that mailbox one at a time, strictly in order. This single property — one message processed at a time, per actor — is what makes an actor's own state immune to race conditions without a single lock anywhere in sight: there is never a moment where two operations are touching that actor's state concurrently, because the mailbox serializes them.

## Isolation as the whole point

Picture a banking system where each account is its own actor. A `transfer` request doesn't reach into two `Account` actors and adjust their balances directly — it sends a `Withdraw` message to one and, once that succeeds, a `Deposit` message to the other. Each account actor processes its own messages one at a time, so there's no way for two simultaneous transfer requests touching the same account to corrupt its balance, even with thousands of transfers happening across thousands of accounts at once across many CPU cores. The concurrency is real and the throughput can be very high — different actors genuinely run in parallel — but *within* any one actor, everything is sequential and therefore simple to reason about. This is the actor model's core trade: you give up direct, synchronous access to another unit's state, and in exchange you get to stop thinking about locks entirely.

## Supervision and "let it crash"

Because actors can create other actors, a natural hierarchy emerges where a parent actor supervises the children it created. Erlang's famous "let it crash" philosophy leans on this directly: rather than writing defensive code in every actor to anticipate and recover from every possible failure, an actor that hits a situation it can't handle is simply allowed to crash, and its supervisor — watching for exactly that — restarts it in a known-good state. This turns fault tolerance into an architectural property of the actor hierarchy rather than something every individual piece of code has to get right through careful exception handling. A telecom switch built this way (Erlang's original use case) can keep routing calls even while individual connection-handling actors are crashing and restarting around the clock, because no single actor's failure takes down any other actor's state.

## What you give up

The isolation is not free. Most actor systems make no guarantee that a message you send will arrive, or arrive in the order you sent it relative to other messages from other senders — contrast this with a normal function call, which either returns a result or throws, with no in-between state of "maybe it happened, you'll find out later." This pushes failure handling — timeouts, retries, acknowledgment messages, idempotency — to the front of the design rather than letting it be an afterthought, much the way a distributed system built from independent network-separated services has to (see `system-design`'s `circuit-breakers-and-retries` and `idempotent-apis-and-exactly-once-semantics` for the same problem one layer further out, between services rather than between in-process actors). The actor model doesn't eliminate the hard parts of concurrent and distributed systems; it relocates them to an interface — message passing — that is far easier to reason about and test in isolation than a tangle of shared memory and locks.
