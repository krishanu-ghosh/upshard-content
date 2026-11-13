---
title: "Synchronous vs. Asynchronous API Design"
description: When to make a client wait for a result versus accept it and let them check back later — the 202 Accepted plus status-polling pattern, and when to notify.
roadmap: api-design
node: synchronous-vs-asynchronous-api-design
group: real-time-and-integration-patterns
video:
  provider: youtube
  id: "U40HzU_KkDY"
  start: 0
  end: null
creator:
  name: Milan Jovanović
  channel_url: https://www.youtube.com/@MilanJovanovicTech
duration_seconds: 548
level: intermediate
prerequisites:
  - webhooks-vs-polling
takeaways:
  - "A synchronous endpoint makes a promise: the response body, when it arrives, is the actual result. An asynchronous endpoint makes a different, weaker promise: the response is an acknowledgment that work has started, and the result comes later through a separate channel — conflating the two, by having clients treat `202 Accepted` as if it were `200 OK` with a slow body, is the single most common async-API bug."
  - "`202 Accepted` means exactly \"I've accepted this and will work on it,\" not \"it succeeded\" — the actual outcome, success or failure, is only known once the asynchronous work finishes, and the response has to give the client something to check next: a URL for the job's status, at minimum."
  - "The status resource a client polls needs its own small, stable contract: a job id, a state (queued, running, succeeded, failed, at minimum — not just a boolean), timestamps for when it was accepted and last updated, and on success a link to the real result resource rather than inlining it, since the first version of the job may run long before the result shape is final."
  - "Sending a `Retry-After` header on the status response (or documenting a client-side backoff policy) turns \"poll until done\" from a guess into a contract — without it, API consumers invent their own polling interval, and you end up serving a wall of status checks from clients polling every 100ms for a job that takes ten minutes."
  - "Whether to make an operation synchronous or asynchronous should track how long it actually takes and how often it fails transiently, not how it's implemented internally — a call that's synchronous today because it usually completes in 200ms still needs an async escape hatch if a slow downstream dependency can occasionally push it past a client's timeout."
  - "`webhooks-vs-polling`, the prior lesson in this group, is the other half of this decision: once a client has a long-running job's status URL, the API can either make them keep polling it or, if they registered a callback, push a webhook when the job finishes — offering both and letting the client pick is usually better product design than forcing one pattern on every consumer."
check_questions:
  - q: "A client gets back `202 Accepted` from a POST that kicks off a long-running report generation job. What should the client do next, and what should it never assume?"
    a: "It should follow the status URL the response provides and poll (or wait for a webhook, if it registered one) to learn the actual outcome; it should never assume the 202 itself means the report succeeded or even that it will succeed — 202 only confirms the request was accepted for processing, and the job can still fail once it actually runs."
  - q: "Why does a well-designed status-polling endpoint return a job state like queued/running/succeeded/failed instead of just a boolean `done: true/false`?"
    a: "A boolean can't distinguish \"still working\" from \"failed and never going to finish,\" so a client polling on `done` alone would poll forever against a job that already failed; an explicit state lets the client stop polling and handle failure as soon as it happens, and a `queued` vs `running` distinction also lets the client (or an operator) tell the difference between a job that hasn't started and one that's stuck."
  - q: "What problem does a `Retry-After` header (or an equivalent documented polling interval) solve that leaving polling frequency undocumented doesn't?"
    a: "Without it, every API consumer picks their own polling interval by guesswork, and some will poll far more aggressively than the job actually needs — generating load on the status endpoint proportional to how impatient each client's code happens to be, rather than to how long the job actually takes; a server-supplied interval turns that into something the API controls directly."
  - q: "Why is \"this endpoint usually responds in 200ms\" not a good enough reason to keep it fully synchronous forever?"
    a: "A usually-fast operation can still have a long tail caused by a slow downstream dependency, a cold cache, or load spikes, and a purely synchronous design has no way to handle that tail except making the client wait past its timeout or fail outright; designing an async escape hatch (or a hybrid that waits briefly, then falls back to 202) means the rare slow case degrades gracefully instead of becoming an outage for anyone unlucky enough to hit it."
deeper:
  - label: "Avoiding long running HTTP API requests."
    url: "https://www.youtube.com/watch?v=2yUnY61zdAk"
related_formulas: []
---

## The promise a response status makes

Every API response makes an implicit promise about what just happened, and the client's entire error-handling and retry logic depends on that promise being accurate. A `200 OK` with a body says: this is the result, right now. A `202 Accepted` says something much weaker: I've taken custody of this request and will work on it — the result, success or failure, is still unknown. Those are different contracts, and the most common failure in asynchronous API design is a client (or, just as often, the API's own documentation) treating them as the same thing with a longer wait. If your API returns `202` for an operation that can fail, and the only way a client finds out it failed is a support ticket three days later, you haven't actually designed an asynchronous API — you've designed a synchronous one that lies about its own completion.

## The shape of the pattern

The core pattern is simple to state and easy to get wrong in the details: the client submits a request, the server validates what it can cheaply validate up front, enqueues the real work, and immediately returns `202 Accepted` with a URL pointing at a status resource for that job. The client (or, per `webhooks-vs-polling`, a registered callback) finds out what happened by checking that URL later. The parts that separate a contract clients can actually build against from a half-finished version of the pattern:

- **The status resource has its own stable shape.** At minimum: a job id, an explicit state — `queued`, `running`, `succeeded`, `failed`, not a bare boolean — and timestamps for when the job was accepted and last updated. A boolean `done` flag can't distinguish "still working" from "failed permanently," which leaves a client with no way to know it should stop polling and handle an error.
- **The result is a link, not an inline body, until it's ready.** On success, the status resource should point at the actual result resource (`GET /reports/abc123`) rather than trying to inline a result that might be large, or whose shape wasn't fully settled when the job started. This also means the result can be fetched, cached, and shared independently of the job that produced it.
- **Failure is a first-class, inspectable state**, not silence. A failed job's status response should say why, at whatever level of detail is safe to expose — the same discipline `structured-error-handling-and-rfc-7807` asks of synchronous error responses applies here, just delivered on a delay instead of in the original response.
- **Polling frequency is part of the contract, not a guess.** A `Retry-After` header on the status response, or a documented minimum polling interval, keeps every client from independently inventing its own interval — which in practice means some fraction of them hammering the status endpoint every hundred milliseconds for a job that reliably takes minutes.

## Designing for eventual completion, not just eventual response

"Asynchronous" describes when the client learns the outcome, not whether the outcome is guaranteed. A well-designed async API has to answer the harder question: what happens to a job that's accepted and then the worker that would process it crashes, or the queue backs up for an hour, or the job depends on a downstream system that's down? The honest answers — retried automatically up to some limit, held in a `queued` state the client can see is not progressing, or moved to a terminal `failed` state with a reason — all have to be real, visible states a client can observe through the status resource, not failure modes that only show up as a job that silently never updates. A status resource that just stops changing, with nothing distinguishing "about to finish" from "orphaned," pushes the same kind of blind debugging onto your API consumers that an unversioned, undocumented webhook retry policy pushes onto webhook receivers.

## When to make the call synchronous at all

Not every operation needs this pattern, and reaching for it by default adds real complexity: an extra resource type, a polling or callback story, and a job-tracking system the status endpoint reads from. The right trigger is how long the operation actually takes and how often that duration has a long tail. An operation that reliably completes in under a second or two is a fine synchronous endpoint — `200`, body, done. The moment that same operation can occasionally take 30 seconds because of a slow downstream call, a large input, or contention, you have a choice to make explicit rather than let happen by accident: either the client eats an occasional timeout (bad), or you build the async escape hatch. A common, pragmatic middle ground — covered in more depth in this lesson's video — is to wait briefly server-side (tens of milliseconds) and return the full result synchronously if it's ready in time, falling back to `202` and a status URL only when it isn't. That gives the fast, common case a simple synchronous contract while still having a real answer for the slow tail, instead of forcing every caller to handle async machinery for an operation that's usually instant.

## Poll, or tell them when it's done

Once a job has a status URL, the API has a second decision to make, and it's the one `webhooks-vs-polling` covers from the delivery side: does the client have to keep asking, or can the API notify them? Offering both — a status endpoint for clients that can't or won't register a callback, and an optional webhook for ones that can — is usually better product design than forcing a single pattern on every integrator. A batch import job that takes twenty minutes is a bad fit for aggressive client-side polling and a good fit for "call me back when it's done"; a quick five-second job someone's UI is actively waiting on is often simpler to just poll twice and be done. `batch-processing-apis`, next in this group, builds directly on this pattern for the specific case of very large batch operations that are async almost by necessity.
