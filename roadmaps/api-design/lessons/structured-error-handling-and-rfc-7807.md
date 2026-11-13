---
title: "Structured Error Handling and RFC 7807"
description: The Problem Details JSON shape for HTTP API errors, and how to design a consistent, machine-readable error contract instead of ad hoc error bodies.
roadmap: api-design
node: structured-error-handling-and-rfc-7807
group: contract-design
video:
  provider: youtube
  id: "eN4GX5WW87s"
  start: 0
  end: null
creator:
  name: Milan Jovanović
  channel_url: https://www.youtube.com/@MilanJovanovicTech
duration_seconds: 974
level: intermediate
prerequisites:
  - cors-and-content-negotiation
takeaways:
  - "RFC 7807 (updated by RFC 9457) defines Problem Details, a standard JSON shape for HTTP API errors with five core members — `type`, `title`, `status`, `detail`, `instance` — served as `application/problem+json`, so clients can parse any compliant API's errors with one deserializer instead of reverse-engineering a bespoke shape per API."
  - "`type` is a URI identifying the specific error category (ideally dereferenceable to human-readable docs, but doesn't have to be), `title` is a short, stable, human-readable summary of that type, `detail` is the specific, this-occurrence explanation, and `instance` optionally identifies exactly which request or resource triggered it — conflating `title` and `detail` is the most common misuse, since `title` should stay constant across every occurrence of that error type while `detail` varies per occurrence."
  - "The HTTP status code and the Problem Details body are not redundant — the status code is what generic HTTP infrastructure (caches, proxies, monitoring) understands without any awareness of your API's error vocabulary, while the body gives an API-aware client the specific, actionable detail the status code's coarse granularity can't carry on its own."
  - "An application-specific error code (a stable string like `INSUFFICIENT_FUNDS` or `ORDER_ALREADY_SHIPPED`) belongs alongside the HTTP status code, not instead of it — the status code says 'this is a client error' or 'this is a conflict' in HTTP's own vocabulary; the error code says exactly which specific condition occurred in your domain's vocabulary, and client code should generally branch on the error code, not by string-matching `detail`."
  - "A validation failure spanning multiple fields needs its own documented extension to the base Problem Details shape — commonly an `errors` array or object mapping field names to per-field messages — because the base RFC format has no native concept of 'more than one thing was wrong at once,' and Problem Details is explicitly designed to be extended with custom members rather than fought against."
  - "This lesson is about the shape of the error a client receives, not what the server does when a downstream dependency is unreliable — retry logic and circuit breakers are infrastructure-level resilience patterns covered in `system-design`'s `circuit-breakers-and-retries`; a well-designed error contract and a well-designed resilience strategy are complementary, not the same concern."
check_questions:
  - q: "In a Problem Details response, what's the difference between `title` and `detail`, and why does conflating them cause problems?"
    a: "`title` is meant to be a short, stable summary of the error *type* that stays the same across every occurrence — \"Insufficient Funds\" — while `detail` is the specific explanation for *this* occurrence — \"Account ending in 4421 has a balance of $12.50, which is less than the requested withdrawal of $50.00.\" Putting occurrence-specific data into `title` means two different failures of the same underlying error type produce different `title` values, which breaks any client logic (or logging/aggregation) that groups or keys on `title` as if it were stable."
  - q: "Why isn't the HTTP status code alone enough, and why isn't a Problem Details body alone enough either?"
    a: "The status code alone can't distinguish between the dozen different reasons a request might be a `422 Unprocessable Entity` — insufficient funds, an expired coupon, a quantity exceeding stock all collapse into the same code. A Problem Details body alone, without a correct status code, breaks anything in the HTTP stack that only looks at the status — caches, monitoring dashboards, generic retry middleware — none of which parse response bodies. The two work together: status code for infrastructure, body for the application."
  - q: "Why should a client's error-handling logic branch on an application-specific error code rather than on the `detail` string?"
    a: "`detail` is meant to be human-readable prose that can change wording, add context, or get localized without being a contract-breaking change — if client code parses it with string matching, any copy edit silently breaks that client. An error code like `INSUFFICIENT_FUNDS` is meant to be a stable, documented identifier specifically so client code has something safe to switch on, the same way a response field name is safe to depend on but the human-readable label next to it in a UI is not."
  - q: "How would you extend the base Problem Details shape to report that three different fields failed validation on the same request?"
    a: "Add a custom member beyond the five standard ones — commonly `errors`, either as an array of `{field, message}` objects or an object keyed by field name — documented as part of your API's Problem Details profile. The base `detail` field can still carry an overall summary (\"3 validation errors occurred\"), while `errors` carries the structured, per-field breakdown a form-rendering client needs to show the right message next to the right input."
deeper:
  - label: "Representing Problem Details in HTTP APIs: An Introduction to RFC 7807"
    url: "https://www.youtube.com/watch?v=UNdUjBqsUqg"
related_formulas: []
---

## Why ad hoc error bodies are worse than they look

Every API returns errors eventually, and without a deliberate convention, teams tend to reinvent the error shape per endpoint, or per developer, as they go: one endpoint returns `{"error": "not found"}`, another returns `{"message": "Not Found", "code": 404}`, a third returns a bare string. None of these are *wrong* in isolation, but a client integrating against the whole API has to write bespoke parsing logic per endpoint, because there's no shared contract for what an error even looks like. This is the same problem `api-versioning-strategies` names for success responses — every field is a promise — except it's easier to overlook on the error path, because errors feel like an afterthought compared to the "real" response shape. They aren't; a client has to handle both equally often, arguably the error path more carefully, since it's the path where getting the response wrong has the worst consequences (silently proceeding as if a request succeeded, or surfacing a raw stack trace to an end user).

**RFC 7807**, "Problem Details for HTTP APIs" (now updated by RFC 9457, which keeps the same core shape and media type), exists to stop every API from solving this problem slightly differently. It defines one JSON shape, served with the `application/problem+json` media type, that any compliant API uses for any error, so a client — or a generic piece of tooling, like an API gateway's error-logging middleware — can parse one shape instead of n bespoke ones.

## The actual shape

A Problem Details object has five standard members, all optional in the strict sense (the spec permits omitting any of them) but conventionally all present:

```json
{
  "type": "https://api.example.com/problems/insufficient-funds",
  "title": "Insufficient Funds",
  "status": 422,
  "detail": "Account ending in 4421 has a balance of $12.50, which is less than the requested withdrawal of $50.00.",
  "instance": "/accounts/4421/withdrawals/9981"
}
```

`type` is a URI identifying the error's *category* — ideally one that resolves to human-readable documentation, though the spec doesn't require it; `about:blank` is the defined default when a more specific type isn't warranted. `title` is a short, human-readable summary of that *type*, and critically, it's meant to stay constant across every occurrence — it describes the kind of problem, not this specific instance of it. `status` repeats the HTTP status code in the body, useful for any consumer that only has the parsed JSON in hand (a webhook retry queue storing the body, say). `detail` is the occurrence-specific explanation — the thing that actually varies between two failures of the same type. `instance` is an optional URI identifying the specific request or resource involved, useful for tracing a reported error back to the request that produced it.

The most common misuse is collapsing `title` and `detail` into one field, usually by putting occurrence-specific data into `title`. If `title` becomes "Account 4421 has insufficient funds for a $50.00 withdrawal," it's no longer a stable identifier for the error type — logging, aggregation, or any client code that keys off `title` as if it were an enum value breaks the moment the specific numbers change, because every occurrence now produces a technically-different title.

## Status code and body are not redundant with each other

It's tempting to treat `status` inside the body as pointless duplication of the HTTP status line, but the two serve different audiences. The status code is what generic infrastructure understands without any awareness of your API's error vocabulary — a CDN deciding whether to cache a response, a monitoring system alerting on 5xx rates, a load balancer's health check. None of that parses response bodies, so a wrong status code (returning `200 OK` with an error buried in the JSON, a mistake more common than it should be) makes all of it silently misbehave. The body, meanwhile, is for an API-aware client that needs more than "this failed" — it needs to know *why*. Get the status code right first, as the coarse signal every layer of the stack can act on, then use the body for the fine-grained detail only your own client code understands.

## Application error codes, and why clients should branch on them

HTTP status codes are coarse by design — `422 Unprocessable Entity` covers an enormous range of actual failure reasons across any real API (insufficient funds, an expired coupon, a quantity exceeding available stock, a business rule violation specific to your domain), and a client that needs to behave differently per reason needs something more specific to branch on than the status code alone provides. The common pattern is a stable, documented application error code — a short string like `INSUFFICIENT_FUNDS` or `ORDER_ALREADY_SHIPPED` — carried as a custom extension member alongside the five standard Problem Details fields (the RFC explicitly permits extending the object with additional members beyond the defined five).

This matters as a design decision, not just a convenience: it's about what client code should be allowed to depend on. `detail` is prose — free to improve its wording or get localized without that counting as a breaking change. An error code is a contract the way a field name is: code that does `if (error.code === "INSUFFICIENT_FUNDS")` depends on something your team has committed to keeping stable, the same discipline `api-versioning-strategies` applies to the rest of the response shape. Code that instead does `if (error.detail.includes("insufficient"))` has built something a routine copy edit will silently break.

## Validation errors need their own shape

A single Problem Details object naturally represents one problem, but a form submission failing validation on three fields at once doesn't fit that model cleanly — there isn't one `detail` string that usefully represents three unrelated failures without a client having to re-parse prose. The RFC's answer is the same extensibility mechanism used for error codes: add a custom member, commonly `errors`, carrying the structured per-field breakdown:

```json
{
  "type": "https://api.example.com/problems/validation-error",
  "title": "Validation Failed",
  "status": 400,
  "detail": "3 validation errors occurred.",
  "errors": [
    { "field": "email", "message": "must be a valid email address" },
    { "field": "password", "message": "must be at least 8 characters" },
    { "field": "username", "message": "is already taken" }
  ]
}
```

`detail` stays useful as a quick human summary, while `errors` gives a form-rendering client exactly what it needs to surface each message next to the right input. The specific shape of `errors` (array of objects vs. an object keyed by field name) is a choice to make once and document as part of your API's own Problem Details profile, then apply consistently everywhere validation errors occur — the value of standardizing on RFC 7807 in the first place comes from never having a second, different validation-error shape show up somewhere else in the same API.

One explicit non-goal, worth naming since it's easy to conflate: this lesson is about the *shape* of an error response a client receives, not about what a server does internally when a downstream dependency is failing or slow. Retry logic, exponential backoff, and circuit breakers are resilience mechanisms that operate beneath the API layer, and `system-design`'s `circuit-breakers-and-retries` covers that ground in full — a well-designed error contract and a well-designed resilience strategy solve different problems and compose cleanly, but neither substitutes for the other.
