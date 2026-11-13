---
title: "Batch Processing APIs"
description: Designing bulk endpoints that accept many items in one request — request/response shape, partial-failure semantics, and when a batch becomes its own async job.
roadmap: api-design
node: batch-processing-apis
group: real-time-and-integration-patterns
video:
  provider: youtube
  id: "PQ5n8R1HZ5Q"
  start: 0
  end: null
creator:
  name: David Anderson
  channel_url: https://www.youtube.com/@DX7Dev
duration_seconds: 2752
level: intermediate
prerequisites:
  - synchronous-vs-asynchronous-api-design
takeaways:
  - "A batch endpoint accepting 100 items and having 3 fail is not an edge case to handle later — it's the normal, expected outcome of any batch operation touching independent items, and the response shape has to be designed around it from the start, not bolted on after the first confused support ticket."
  - "The two honest options for batch semantics are all-or-nothing (the whole batch succeeds or none of it does, inside one transaction) and per-item/best-effort (each item succeeds or fails independently, and the response reports each outcome) — a batch API that doesn't explicitly declare which one it is, in its documentation and in its response shape, leaves every caller to guess and get it wrong."
  - "A per-item response needs, for every item submitted: the caller's own correlation key (so results can be matched back to requests without relying on array position), a status, and either the created/updated resource or a structured error in the same shape `structured-error-handling-and-rfc-7807` uses for single-item failures — array position alone is a fragile correlation mechanism the moment retries or reordering enter the picture."
  - "HTTP's status-code vocabulary wasn't built for \"207 out of 212 succeeded\": `207 Multi-Status` exists for exactly this and is the technically correct choice, but plenty of production APIs return a plain `200` with per-item statuses in the body instead, because some HTTP clients, proxies, and monitoring tooling handle 207 poorly — pick one deliberately and document it, rather than it becoming an accident of whichever framework default was easiest."
  - "Retrying a partially-failed batch without duplicating the items that already succeeded needs the same idempotency discipline as any other write: a batch-level idempotency key for the whole submission, and ideally a per-item key too, so a client can safely resubmit a batch (or just the failed items) after a timeout without re-creating what already landed."
  - "A batch large enough to risk exceeding a request timeout (thousands of items, a bulk file upload, anything measured in minutes rather than milliseconds) stops being a single-request problem and becomes the async job pattern from `synchronous-vs-asynchronous-api-design`: accept the batch, return a job id, and let the client poll or get notified for a result that itself reports per-item outcomes once the job completes."
check_questions:
  - q: "A client submits a batch of 50 items to create, and 47 succeed while 3 fail validation. In a well-designed per-item batch API, what should the HTTP response look like, and why is returning a single error for the whole request the wrong move?"
    a: "The response should report an outcome for all 50 items — the 47 successes with their created resources (or links to them) and the 3 failures with structured, per-item errors — typically under `207 Multi-Status` or a `200` with a results array, whichever the API has documented; returning one error for the whole batch would force the client to assume everything failed and either lose the 47 successful creations or resubmit the entire batch and risk duplicating them."
  - q: "Why is array position alone a poor way to match batch response items back to the request items a client submitted?"
    a: "If the API retries internally, reorders items for processing efficiency, or the client itself resubmits a partial batch after a failure, positional matching silently breaks and results get attributed to the wrong request item; a caller-supplied correlation key on each item, echoed back on the matching result, stays correct regardless of ordering or retries."
  - q: "What's the core difference between all-or-nothing and per-item batch semantics, and what does an API have to do either way?"
    a: "All-or-nothing wraps the whole batch in one transaction so it either fully succeeds or fully rolls back, which is simpler to reason about but means one bad item blocks 99 good ones; per-item semantics process each item independently and report individual outcomes, which is more forgiving but requires a response shape that can express mixed results — either way, the API has to pick one explicitly and say so, rather than leaving callers to infer it from behavior."
  - q: "When does a batch endpoint stop being a single synchronous request and need to become an asynchronous job instead?"
    a: "Once the batch is large enough, or individual items slow enough, that total processing time risks exceeding a reasonable request timeout — thousands of items, large file-based bulk uploads, or anything that could take minutes rather than milliseconds — at which point the right design accepts the batch, returns a job id per the `202 Accepted`/status-polling pattern, and reports per-item success and failure once that job completes rather than holding the client's connection open the whole time."
deeper:
  - label: "Everything You Need To Know About The Bulk API 2.0"
    url: "https://www.youtube.com/watch?v=DGRAIvPCTGE"
related_formulas: []
---

## Batch endpoints exist to amortize overhead, not to hide failure

The pitch for a batch endpoint is straightforward: instead of making a client issue 200 individual `POST /orders` calls — paying connection overhead, auth checks, and rate-limit accounting 200 times — let them submit all 200 in one request. That's a real and valuable optimization. The part that gets designed badly, over and over, is what happens when some of those 200 items are fine and some aren't. A single-item endpoint has a simple failure model: the request succeeded or it returned an error. A batch endpoint touching 200 independent items essentially never has a single boolean outcome, and an API that's only ever been tested with all-succeed or all-fail inputs will ship a response shape that falls apart the moment real traffic — which always includes some invalid rows, some duplicate keys, some items hitting a business rule the other 199 didn't — hits it.

## Pick a semantics and say so

There are exactly two honest answers to "what happens when some items in a batch fail," and the API has to commit to one of them as a documented, stable contract:

- **All-or-nothing (transactional).** The whole batch succeeds or none of it does — typically by wrapping the batch in a single database transaction and rolling back on any failure. This is simpler to reason about (the client never has to handle mixed outcomes) and is the right choice when the items in a batch are actually related — a multi-line accounting entry that has to balance, a set of inventory adjustments that have to stay consistent with each other. Its cost is that one malformed item among a thousand good ones blocks the whole submission.
- **Per-item (best-effort).** Each item is processed independently, and the response reports an outcome for every single one. This is the right default when items are genuinely independent — a bulk contact import, a batch of notification sends — and it's far more forgiving of partial bad data, at the cost of a response shape that has to express "mostly succeeded" instead of a single status code.

The failure mode to avoid isn't picking the "wrong" one of these — both are legitimate, and the right choice depends on whether your items are actually coupled — it's not picking either one explicitly. An API whose batch behavior is just "whatever the implementation happens to do when it hits an error" leaves every integrator to discover the real semantics by testing failure cases in production, which is exactly the kind of undocumented behavior `structured-error-handling-and-rfc-7807` exists to eliminate for single-item errors, now multiplied across however many callers build retry logic on top of a guess.

## The response shape that makes per-item results usable

If you've chosen per-item semantics, the response needs more structure than "an array of results in the same order you sent them." At minimum, each result needs:

- **The caller's own correlation key**, echoed back — a client-supplied id on each submitted item, not just its position in the array. Position-based matching looks fine until a retry, an internal reordering for processing efficiency, or a partial resubmission enters the picture, at which point it silently attributes results to the wrong items.
- **An explicit per-item status** — not just presence-or-absence of an error field, which is easy to get wrong when an item "succeeds" with a null or empty result.
- **Either the resulting resource (or a link to it) or a structured error**, in the same error shape your single-item endpoints use. There's no reason a batch failure should carry a different, bespoke error format than the same validation failure would produce on `POST /orders/one-at-a-time` — consistency here is what lets a client reuse its existing error-handling code instead of writing a second, batch-specific parser.

On the outer response, `207 Multi-Status` is the technically precise HTTP status for "some succeeded, some didn't," and it exists in the spec for exactly this situation. In practice, plenty of well-run batch APIs return a plain `200` with the per-item breakdown in the body instead, because some HTTP client libraries, proxies, and monitoring dashboards handle `207` inconsistently or treat it as unexpected. Either choice is defensible; what's not defensible is leaving it unstated and letting callers infer your batch status-code convention from trial and error.

## Idempotency, now per-item as well as per-batch

A client that submits a batch, experiences a timeout, and doesn't know whether the batch landed faces the exact problem `webhooks-vs-polling` solves for webhook receivers, just from the client's side of a request instead of the server's side of a delivery: safely retrying without duplicating work. A **batch-level idempotency key** on the whole submission handles the simple case — resubmitting the identical batch with the same key returns the original result instead of reprocessing. The sharper version of the problem is a batch that partially succeeded before the timeout: resubmitting the whole batch would re-create the items that already landed unless each item also carries its own idempotency key the server can check individually. Designing for this up front — a `client_item_id` or similar on every batch item, checked against what's already been processed — turns "did my batch actually go through?" from an open question into something the client can safely resolve by just resubmitting.

## When a batch becomes a job

Everything above assumes the batch finishes within one request's reasonable timeout. Once it doesn't — thousands of items, a bulk CSV or JSONL upload, anything that could plausibly take minutes — the right design stops being "one big synchronous request" and becomes the asynchronous pattern from `synchronous-vs-asynchronous-api-design`: accept the batch, validate what can be validated cheaply up front, return `202 Accepted` with a job id, and let the client poll a status endpoint (or receive a webhook) for completion. The result that status endpoint eventually returns is still, fundamentally, the same per-item success/failure report this lesson has been building toward — the only thing that changes is *when* the client learns it, not the shape of what they learn. Treat "is this batch synchronous or does it need to be a job" as a question with an actual threshold your API documents (item count, payload size, or an estimated processing time), not a judgment call that quietly changes as your system's load profile shifts under you.
