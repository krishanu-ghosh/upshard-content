---
title: "API Versioning Strategies"
description: How to signal API versions to clients, what actually counts as a breaking change, and how to retire an old version without an outage.
roadmap: api-design
node: api-versioning-strategies
group: contract-design
video:
  provider: youtube
  id: "a4KIBU9UgZ4"
  start: 0
  end: null
creator:
  name: LearnThatStack
  channel_url: https://www.youtube.com/@LearnThatStack
duration_seconds: 403
level: intermediate
prerequisites:
  - rest-resource-modeling-and-uri-design
takeaways:
  - "Every field you return is a promise to existing clients — versioning exists because you will eventually need to break that promise, and the question is never whether to version but how to signal the break and how long to honor the old promise."
  - "URI versioning (`/v1/orders`) is the most visible and cacheable option but clutters every path and multiplies maintained surface area; header versioning (a custom header or an `Accept` media-type parameter) keeps URIs stable but is invisible to casual inspection and harder for intermediary caches and proxies to key on."
  - "Additive changes — a new optional field, a new endpoint, a new optional query parameter — are not breaking and don't need a version bump; removing a field, renaming one, changing a field's type, or changing default behavior are breaking and do need one, which is the single most useful filter for deciding whether to version at all."
  - "A deprecation policy without an enforcement mechanism is a suggestion — the `Deprecation` and `Sunset` HTTP response headers turn \"we'd like you to migrate\" into a machine-readable, monitorable signal clients and your own dashboards can both act on."
  - "How long you support an old version is a business decision disguised as a technical one — six to twelve months of overlap is a common default, but the real input is how much control you have over your actual client base, not a rule from a blog post."
check_questions:
  - q: "Why doesn't adding a new optional field to an API response require a new version?"
    a: "Existing clients that don't know about the field simply ignore it — their deserialization and logic are unaffected — so nothing that depended on the old contract breaks. A version bump is for changes that existing, unmodified clients can't safely absorb."
  - q: "What's the main practical trade-off between URI versioning and header versioning?"
    a: "URI versioning is visible in logs, browser bars, and cache keys, and it's trivial to route differently per version at the infrastructure layer, at the cost of cluttering every path and conceptually implying \"a different resource\" for what's really the same resource evolving. Header versioning keeps one canonical URI per resource but makes the version invisible without inspecting headers, and complicates caching and debugging through intermediaries that don't key on header values."
  - q: "What do the `Deprecation` and `Sunset` headers each communicate, and why are both useful?"
    a: "`Deprecation` tells a client \"this version is deprecated as of this date\" (a warning, not a cutoff), while `Sunset` gives the actual date the version stops being served (a deadline). Sending both lets a client's tooling distinguish \"you should start planning\" from \"you have until this date,\" rather than conflating a warning with an ultimatum."
  - q: "A team changes an endpoint's error response from a plain string message to a structured JSON object. Is this breaking?"
    a: "Yes — any client parsing the old plain-string shape will fail or behave incorrectly against the new structured shape, even though the endpoint \"still returns an error.\" Changing a field's type or structure is breaking regardless of whether the field's purpose (communicating an error) stayed the same."
deeper:
  - label: "Master API Versioning: 4 Key Strategies You Need to Know"
    url: "https://www.youtube.com/watch?v=AkKGcJ7FWrM"
related_formulas: []
---

## Why versioning is unavoidable

An API's response shape is a contract, whether or not anyone wrote it down as one. The moment a client goes live against your endpoint, every field it reads, every status code it branches on, and every assumption it makes about ordering or defaults becomes something you're implicitly promising to keep stable. Software evolves, requirements change, and eventually you'll need to change something a real client depends on. Versioning is the mechanism for making that change without breaking everyone who hasn't opted into it yet. The goal isn't to avoid ever breaking anything — that's not achievable over a long enough timeline — it's to make breaks explicit, opt-in, and bounded in time rather than silent and immediate.

This lesson assumes you've already settled on REST as your style (`rest-resource-modeling-and-uri-design` covers the resource/URI design that precedes this) and picked an architectural approach (`system-design`'s `api-architectural-styles` covers REST vs GraphQL vs gRPC at the conceptual level, if that decision is still open). Versioning strategy is a layer on top of both.

## Where the version lives

Three places can carry a version signal, and each has a real cost.

**URI versioning** puts the version in the path: `/v1/orders`, `/v2/orders`. It's the most visible option — anyone reading a log line or a bug report can see which version was hit without inspecting headers — and it plays well with HTTP caching and with infrastructure-level routing, since a reverse proxy or load balancer can route `/v2/*` to an entirely different backend deployment with zero application code involved. The cost is that it implies `/v1/orders` and `/v2/orders` are different resources, when conceptually they're the same resource at different points in its evolution, and every version you keep alive multiplies the number of paths your documentation, your gateway rules, and your test suite have to track.

**Header versioning** keeps one canonical URI and puts the version in a custom request header (`X-API-Version: 2`) or, more purist, in the `Accept` header's media type parameter (`Accept: application/vnd.myapi.v2+json`). This is the REST-ideal answer to "the URI identifies the resource, not its representation format" — the resource is still `/orders`, and the version is metadata about which representation you want back, which is exactly what content negotiation is for. `cors-and-content-negotiation`, later in this group, covers the `Accept`/`Content-Type` mechanics this relies on. The trade-off is discoverability and caching: the version is invisible to anyone just glancing at a URL, intermediary caches that key purely on URL will serve the wrong version's cached response unless they're configured to vary on the header (`Vary: Accept`), and some HTTP clients and API testing tools make custom headers more awkward to set than a URL path.

**Query-parameter versioning** (`?version=2`) is a third, less-recommended option that gets some of URI versioning's visibility without a clean path structure, and is generally worth avoiding for a primary public API — it reads as an afterthought rather than a deliberate contract.

There's no universally correct choice. Public, developer-facing APIs lean toward URI versioning because of its visibility and because external developers debugging against your API benefit from seeing the version at a glance. Internal service-to-service APIs, where both ends are controlled by teams who can coordinate on tooling, lean more often toward header versioning to keep URIs stable and let infrastructure like a gateway handle the version negotiation. GitHub's API is a well-known header-versioning example; most consumer-facing SaaS APIs default to URI versioning.

## What actually counts as breaking

The decision that matters more than which signaling mechanism you pick is knowing *when* you need a new version at all. A useful filter: **if an existing, unmodified client would misbehave against the new response, it's breaking.** Adding a new optional field to a response is not breaking — old clients ignore fields they don't know about. Adding a new endpoint is not breaking. Adding a new optional request parameter with a sensible default is not breaking. Removing a field, renaming a field, changing a field's type (a string becoming a number, a scalar becoming an object), changing a previously-optional field to required, changing a default value, or changing the meaning of an existing status code — all of these are breaking, because something a working client already does will now fail or silently misbehave.

The subtle case is changing an error shape, which teams often treat as "just error handling" rather than part of the contract. It isn't — `structured-error-handling-and-rfc-7807`, later in this group, covers why a consistent, versioned error contract matters exactly as much as the success-path contract does.

## Retiring an old version without an outage

A version with no retirement plan accumulates forever, and "we'll deprecate it eventually" without a mechanism never actually happens — there's always a reason not to break the next customer this month. The fix is making deprecation a managed, visible process instead of a one-time announcement. Two HTTP response headers exist specifically for this: `Deprecation: true` (or a date) tells a client's tooling "this version is deprecated, you should be planning a migration," and `Sunset: Sat, 31 Jan 2027 00:00:00 GMT` tells it the hard date the version stops being served. Sending both — well before the sunset date, consistently, on every response from the deprecated version — lets responsible API consumers build automated alerts around them rather than relying on a mailing list announcement someone might miss.

How long to keep an old version alive is a business call wearing a technical hat. Six to twelve months of overlap between announcing deprecation and actually shutting a version off is a common default for public APIs with real external dependents, but the right number depends entirely on how much control you have over your callers: an internal API where you control every caller can retire in weeks once they've migrated; a public API with enterprise customers on annual contracts may need a year or more. The one mistake worth naming explicitly: never sunset a version without first instrumenting usage on it. If you can't see which clients are still calling `/v1/orders`, you have no way to know whether shutting it off tomorrow is safe or catastrophic.
