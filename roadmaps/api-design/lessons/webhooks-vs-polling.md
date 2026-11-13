---
title: "Webhooks vs. Polling"
description: Designing webhook delivery a receiver can trust — retry/backoff, HMAC signing, idempotent receipt against at-least-once delivery — and when polling wins.
roadmap: api-design
node: webhooks-vs-polling
group: real-time-and-integration-patterns
video:
  provider: youtube
  id: "FuBSI2I9rAI"
  start: 0
  end: null
creator:
  name: InterviewDryRun
  channel_url: https://www.youtube.com/@InterviewDryRun
duration_seconds: 1393
level: intermediate
prerequisites:
  - streaming-and-real-time-api-contracts
takeaways:
  - "A webhook is at-least-once delivery, never exactly-once: the sender's retry policy means your endpoint must assume every event can arrive more than once, and design for that from the start rather than patching it in after the first duplicate-charge bug."
  - "Delivery order is not guaranteed either — a retried `invoice.paid` can arrive after a fresher `invoice.refunded` for the same invoice — so a receiver that assumes \"later HTTP request means later event\" will eventually process events out of order and get the wrong end state."
  - "Retry/backoff is the sender's job and belongs in the contract, not just the implementation: exponential backoff with jitter, a capped number of attempts or a capped total window (hours, not minutes), and a dead-letter state the receiver can inspect and manually replay from, because an event that silently stops retrying after attempt five with no record anywhere is a support ticket waiting to happen."
  - "HMAC signature verification proves the payload came from the sender and wasn't altered in transit — compute an HMAC over the raw request body with a shared secret, compare it to a signature header using constant-time comparison, and verify it before doing anything else with the request, including before your JSON parser touches it."
  - "Idempotency on receipt means keying off a stable event id (not the request, not a timestamp) and recording that you've already processed it before doing anything with side effects — a lookup-then-insert on that id, inside the same transaction as the side effect, is what actually makes a duplicate delivery harmless instead of merely unlikely to cause problems."
  - "Polling beats webhooks when the receiver can't expose a reachable public endpoint (desktop apps, devices behind NAT, locked-down corporate networks), when near-real-time isn't actually required, or when the integration is low-volume enough that building and operating webhook infrastructure — endpoint registration, retries, signature verification, a replay UI — costs more than it saves; it's a legitimate choice, not a fallback for teams that couldn't be bothered."
check_questions:
  - q: "Why must a webhook receiver be built to handle duplicate deliveries of the same event, rather than treating duplicates as a rare edge case?"
    a: "Webhook delivery is at-least-once by design — if the receiver's endpoint times out, returns a 5xx, or the sender simply can't confirm receipt, the sender retries the same event — so from the receiver's side, \"the same event id arrives twice\" is a normal and expected occurrence, not a failure mode to patch around later."
  - q: "What's wrong with verifying a webhook's signature after parsing the JSON body instead of before?"
    a: "Signature verification has to run over the exact raw bytes that were signed; re-serializing parsed JSON can change whitespace, key order, or number formatting, which produces a different byte sequence than what was signed and makes a legitimate payload fail verification — and more importantly, acting on the payload (even just parsing it into application types) before confirming it's authentic defeats the point of verifying it at all."
  - q: "A payment processor's webhook fires twice for the same successful charge because the receiver's endpoint was briefly slow and the first attempt timed out before the processor got the response. How does idempotent receipt logic prevent this from charging or crediting the account twice?"
    a: "The receiver records the event's unique id as processed (inside the same transaction as the side effect it triggers, like marking an order paid) before or atomically with doing the work; when the duplicate delivery arrives, the receiver looks up that same event id, finds it already recorded, and returns success without repeating the side effect — the dedup check has to live at the event-id level, not at the HTTP-request level, since the two requests are genuinely separate requests carrying the same logical event."
  - q: "What's a concrete scenario where polling is the better engineering choice over webhooks, and why?"
    a: "A desktop application integrating with a cloud API has no stable, publicly reachable URL for the API to call back to, so it cannot receive webhooks at all regardless of how well-designed the webhook system is — polling a status or changes endpoint on an interval is the only workable option, and for a desktop client checking in every few minutes is usually an acceptable trade against the cost of maintaining a relay/tunneling service just to accept inbound webhook calls."
deeper:
  - label: "Webhook System Design Explained: Verify, Queue, Retry"
    url: "https://www.youtube.com/watch?v=feim7K7rtRM"
related_formulas: []
---

## Webhooks are a server calling your server

A webhook inverts the usual API relationship: instead of your code calling the provider, the provider's server makes an HTTP request to a URL you registered, the moment something happens. That's the appeal — no polling loop, near-instant delivery, no wasted "anything new?" requests — and it's also exactly why webhooks are harder to design correctly than a normal API endpoint. You've handed a piece of your infrastructure's inbound surface to a request initiator you don't control the timing or retry behavior of, and the entire design problem is making that trustworthy: trustworthy that the payload is genuine, trustworthy that duplicates and out-of-order delivery don't corrupt your state, and trustworthy that a temporary outage on your end doesn't silently lose events.

## At-least-once, and never assume ordering

No mainstream webhook system promises exactly-once delivery, and for good reason: the sender genuinely cannot know, from a timeout or a dropped connection, whether your endpoint received and processed the event or not. The only safe default is **at-least-once** — if the sender isn't sure, it retries, and "sure" requires your endpoint to return a success status before the sender considers the event delivered. The corollary receivers routinely get wrong: **delivery order isn't guaranteed either**. A retried request for an older event can land after a fresh request for a newer one on the same resource. If your handler logic is "whatever the latest webhook says, write it," you will eventually overwrite a correct, newer state with a stale, retried one. The fix is to carry an ordering signal in the event itself — a sequence number, or a resource version/timestamp the receiver compares against what it already has — and ignore (or at least don't blindly overwrite with) anything older than what's already recorded.

## Retry and backoff belong in the contract

If you're the one sending webhooks, your retry policy is part of your API's public contract, not an implementation detail you're free to change without telling anyone. A reasonable shape: exponential backoff with jitter (so a burst of failures across thousands of receivers doesn't retry in lockstep and hammer them all at once), a capped total retry window measured in hours — not a handful of attempts over a few seconds — and a terminal **dead-letter** state once that window elapses, with a log a customer can actually see and manually replay from. Receivers build real operational trust in a webhook system by being able to answer "did this fire, and if it failed, can I see why and retry it myself" — a system that just drops the event silently after five attempts, with no record, pushes that failure onto a support queue instead of a self-service replay button.

## Proving the payload is real: HMAC signatures

Because a webhook endpoint is a public, internet-reachable URL, anyone who finds it can send it a forged payload unless you verify where it actually came from. The standard mechanism is an **HMAC signature**: sender and receiver share a secret (issued when the webhook is registered), the sender computes an HMAC (commonly HMAC-SHA256) over the exact raw request body, puts it in a header, and the receiver recomputes the same HMAC over the raw bytes it received and compares. Three details decide whether this actually works:

- **Verify over the raw body, before parsing it.** JSON deserialization and re-serialization can change whitespace, key order, or number formatting; the recomputed HMAC has to run over the identical byte sequence the sender signed, which means capturing the raw body before any JSON middleware touches it.
- **Compare signatures in constant time.** A naive `==` string comparison short-circuits on the first mismatched byte, which leaks timing information an attacker can use to guess the correct signature byte by byte. Use your language's constant-time comparison primitive.
- **Add replay protection.** A valid signature alone doesn't stop someone from capturing a legitimate request and resending it later. Signing a timestamp along with the body, and rejecting anything outside a short tolerance window (a few minutes), closes that gap — this is exactly the kind of trust decision `authorization-models-rbac-abac-and-beyond`, later in this roadmap, treats more generally: proving *who* is asking is necessary but not sufficient; you also need to bound *when* that proof is still valid.

## Idempotency on receipt

Signature verification tells you the payload is genuine. It says nothing about whether you've already processed it. That's a separate mechanism: every event carries a stable, unique event id, and the receiver's handler records that id as processed — atomically with the side effect it triggers, in the same database transaction — before considering the request handled. When the inevitable duplicate arrives (because the first delivery's response got lost even though your handler actually ran), the lookup against that id finds it already recorded and the handler returns success without repeating the side effect. The id to key on is the **event id**, never the HTTP request itself — two separate requests, arriving minutes apart, can carry the same logical event, and request-level dedup (an `Idempotency-Key` header, the pattern used for client-initiated writes) doesn't help you here because you're not the one generating the key.

## When polling is just the better answer

None of the above is free to build, and webhooks aren't automatically the sophisticated choice. Polling wins outright when the receiver has no stable, publicly reachable endpoint to call back to — a desktop app, something behind NAT, a device on a locked-down network — because there is no webhook design, however careful, that works without an inbound address. It also wins when near-real-time delivery genuinely isn't needed and the integration is low-volume enough that the ongoing cost of webhook infrastructure (endpoint registration and health checks, retry and dead-letter handling, signature verification, a replay UI for customers) outweighs the latency you'd save. The `streaming-and-real-time-api-contracts` lesson's backpressure discussion applies in reverse here too: a receiver that occasionally polls a `GET /events?since=<cursor>` endpoint on its own schedule is, in effect, applying its own backpressure for free — it only ever asks for more than it can handle if it chooses to. Treat the choice as what it is: a trade-off between latency and operational simplicity, decided by what your actual client population can support, the same framing `synchronous-vs-asynchronous-api-design`, next in this group, applies to the broader question of when a client should wait for a result versus be told about it later.
