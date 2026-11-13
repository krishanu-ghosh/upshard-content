---
title: "Idempotent APIs and Exactly-Once Semantics"
description: Why retries make idempotency essential, how idempotency keys work, and why "at-least-once plus idempotent" beats chasing true exactly-once.
roadmap: system-design
node: idempotent-apis-and-exactly-once-semantics
group: networking-apis-and-communication
video:
  provider: youtube
  id: "xeBY8fCWfvU"
  start: 0
  end: null
creator:
  name: CodeOpinion
  channel_url: https://www.youtube.com/@CodeOpinion
duration_seconds: 728
level: intermediate
prerequisites:
  - api-architectural-styles
takeaways:
  - A distributed system that retries on timeout can never fully tell the difference between "the request never arrived" and "the request arrived and succeeded, but the response was lost" — so it has to retry, and that retry can duplicate a side effect unless the API is built to handle it.
  - An idempotent operation produces the same end state no matter how many times it's applied, which makes retries safe by construction instead of requiring the caller to guess whether the first attempt actually went through.
  - An idempotency key is a unique identifier the client generates once per logical operation and sends with every retry attempt, letting the server recognize a repeat and return the original result instead of repeating the side effect.
  - True exactly-once delivery is practically unachievable across an unreliable network, because a crash or lost acknowledgment at the wrong instant always leaves either a duplicate or a gap — no amount of protocol cleverness removes that fundamental ambiguity.
  - The realistic target is at-least-once delivery combined with idempotent processing — let the network retry as much as it needs to, and make sure that however many times a message lands, its effect only happens once.
check_questions:
  - q: Why can't a client reliably distinguish between a request that never arrived and one that succeeded but whose response got lost?
    a: In both cases, the client experiences the same symptom — no response within the expected time — so from the client's point of view, a timeout after a successful-but-unacknowledged request looks identical to a timeout from a request that never reached the server, and the client has no way to tell which happened without additional information.
  - q: How does an idempotency key let a server tell a retry apart from a brand-new request?
    a: The client generates one unique key per logical operation and sends it with the original request and every retry of it; the server stores the key alongside the result of the first successful execution, and when it sees that same key again it returns the stored result directly instead of re-running the operation.
  - q: Why is "at-least-once delivery plus idempotent processing" considered the practical answer instead of pursuing true exactly-once delivery?
    a: True exactly-once delivery would require eliminating the ambiguity a network failure creates between "never sent," "sent but not acknowledged," and "sent and acknowledged but the ack was lost," which no protocol can fully resolve — so instead of fighting that, systems accept that messages may arrive more than once (at-least-once) and simply make sure processing a duplicate causes no extra effect.
deeper:
  - label: "Idempotency, a key term in distributed systems"
    url: "https://www.youtube.com/watch?v=QhTj3Ax98sY"
related_formulas: []
---

## Retries are inevitable, and they're dangerous by default

Any system made of more than one machine talking over a network has to deal with requests that time out. A network partition, a slow garbage-collection pause, a load balancer dropping a connection — any of these can make a client wait past its timeout without a response. The client's only real option at that point is to retry: send the request again and hope this attempt completes.

The danger is what that retry might do if the *original* request actually succeeded before the client gave up on it. Consider a "charge $50 to this card" API call that times out from the client's perspective. Maybe the server never received it. Maybe the server received it, charged the card, and the response was lost on the way back. The client has no way to distinguish these two cases — both look exactly like "no response arrived in time." If the client retries blindly and the first attempt actually succeeded, the customer gets charged twice.

## Idempotency: making repetition harmless

An operation is **idempotent** when applying it multiple times produces the same result as applying it once. `UPDATE accounts SET balance = 100 WHERE id = 42` is idempotent — running it five times leaves the balance at exactly 100, same as running it once. `UPDATE accounts SET balance = balance - 50 WHERE id = 42` is not — running it five times subtracts 250, not 50. The entire idea behind building idempotent APIs is to make sure that retrying a request is always safe, because the *effect* of repeating it is identical to the effect of doing it once, regardless of how many times it actually lands.

Some operations are naturally idempotent just by how they're phrased — setting a resource to an absolute value rather than incrementing it, or using `PUT` to replace a resource at a known URL rather than `POST` to append a new one. But plenty of real operations aren't naturally idempotent — "charge this card," "create a new order," "send this email" — and for those, the API needs an explicit mechanism.

## Idempotency keys: how it works in practice

An **idempotency key** is a unique token the *client* generates once per logical operation — typically a UUID — and includes with the request:

```
POST /charges
Idempotency-Key: 8f14e45f-ceea-467e-bd72-9b0c1d3a5a21
{ "amount": 5000, "currency": "usd", "card": "tok_..." }
```

The client sends the exact same key on every retry of that same logical charge. The server, on first seeing a key, processes the request normally and stores the key alongside the result (and often marks the key as "in progress" with a conditional write immediately, before doing the real work, so that two near-simultaneous retries don't both slip through and double-process). On any subsequent request carrying the same key, the server skips re-running the operation entirely and just returns the stored result. The client gets a correct response either way — whether this was the first attempt or the fifth — and the card gets charged exactly once no matter how many times the request was retried.

This only works, of course, if the key is scoped to one logical operation and the client is disciplined about reusing it correctly: a fresh key per retry would defeat the whole mechanism, since the server would see each retry as a brand-new, never-seen-before request. Payment APIs like Stripe's are the canonical example, typically expiring stored keys after roughly 24 hours, long enough to cover realistic retry windows without keeping results forever.

## Why true exactly-once delivery doesn't exist

It's tempting to wish the underlying messaging layer would just guarantee **exactly-once delivery** and make this whole problem disappear. It can't, not fully, and the reason is structural rather than an engineering shortfall. Delivering a message exactly once requires the sender to know, with certainty, that the message was received and processed — but the only way to know that is an acknowledgment, and the acknowledgment itself travels over the same unreliable network and can be lost. If the ack is lost, the sender is right back to not knowing whether to resend, and resending risks a duplicate; not resending risks a silent loss. There is no message or protocol trick that removes this ambiguity — it's inherent to communicating over a channel that can lose or delay messages. (Within a single process, calling a function exactly once is trivial; the problem is specifically about coordination across a boundary that can fail partway through.)

## The real-world answer: at-least-once plus idempotent processing

Given that true exactly-once delivery isn't achievable, the practical strategy flips the problem around: accept that a message or request might be delivered **at least once** — possibly more than once, if there's any doubt — and make sure that doesn't matter, because processing the same message twice has no additional effect beyond processing it once. That's exactly what idempotency keys deliver for request/response APIs, and the identical principle shows up in message consumers: a consumer that records which message IDs it has already processed (or structures its writes to be naturally idempotent, like an upsert keyed by a stable ID) gets the same safety against duplicate deliveries from a queue or event stream.

This reframing — stop trying to prevent duplicates at the delivery layer, and instead make duplicates harmless at the processing layer — is one of the more load-bearing ideas in distributed systems design. It shows up again almost everywhere retries and failures are possible, from message-queue consumers to distributed transactions to the client-retry logic behind nearly every production API.
