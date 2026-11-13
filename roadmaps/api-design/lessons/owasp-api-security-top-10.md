---
title: "The OWASP API Security Top 10"
description: The ten most common ways a real API gets broken into, led by broken object level authorization — a narrower list than the general web-application OWASP Top 10.
roadmap: api-design
node: owasp-api-security-top-10
group: security-performance-and-compliance
video:
  provider: youtube
  id: "sl1yqGhuVy4"
  start: 0
  end: null
creator:
  name: InsiderPhD
  channel_url: https://www.youtube.com/@InsiderPhD
duration_seconds: 1783
level: advanced
prerequisites: []
takeaways:
  - "Broken Object Level Authorization (BOLA) is API1 and the single most common real-world API vulnerability: an endpoint checks that a caller is authenticated but never checks that the specific object they're requesting actually belongs to them, so `GET /orders/123` happily returns someone else's order to anyone who can guess or enumerate the ID."
  - "BOLA, Broken Function Level Authorization (BFLA), and Broken Object Property Level Authorization (BOPLA) are three distinct authorization failures that get collapsed into one worry in most teams' heads — object-level (which record), function-level (which endpoint/role), and property-level (which fields) each need their own check, and passing one says nothing about the other two."
  - "BOPLA merges what used to be two separate 2019 categories — excessive data exposure (the API returns more fields than the client needed, trusting the frontend to hide them) and mass assignment (the API blindly writes every field in a request body to the model, letting a client set `is_admin: true` on their own user object)."
  - "Unrestricted Resource Consumption and Unrestricted Access to Sensitive Business Flows are both about abuse at normal-looking request volume — no single request is malformed, but nothing stops a script from calling an expensive endpoint a million times or buying out limited inventory in milliseconds, because the API has no concept of a rate or a flow being abnormal."
  - "Improper Inventory Management is the risk of the API you forgot you're running — a `v1` endpoint nobody decommissioned, a staging deployment with production data, an internal debug route that shipped to prod — and it matters because every other item on this list only protects the APIs your security review knew to look at."
  - "Unsafe Consumption of APIs flips the usual threat model: your own API can be the victim of a third-party API's response, if you deserialize and trust whatever a webhook or upstream integration sends you without validating it the same way you'd validate client input."
check_questions:
  - q: Why does the general OWASP Top 10's "broken access control" category not fully cover BOLA?
    a: The general list's broken access control is mostly about role- and permission-level checks — can this user reach this endpoint at all — while BOLA is specifically about object-level checks within an endpoint the user is already allowed to call; an API can have perfect role-based access control and still be fully vulnerable to BOLA if it never verifies that the specific `order_id` in the URL belongs to the caller.
  - q: A response includes a `user` object with `email`, `name`, and `ssn_last4` fields, and the frontend only displays the first two. Is this a vulnerability, and which category?
    a: Yes — it's excessive data exposure, half of BOPLA (API3). The API is trusting the client to hide fields it was never supposed to send across the wire at all; anyone inspecting network traffic or calling the endpoint directly gets the SSN fragment regardless of what the UI renders.
  - q: Why can an API with solid authentication and rate limiting still be vulnerable to Unrestricted Access to Sensitive Business Flows?
    a: Rate limiting caps request volume, but a sensitive business flow (buying a limited-drop product, creating accounts to farm a signup bonus) can be abused at a rate well under any reasonable per-IP or per-token limit, especially when spread across many accounts or proxies — the fix is detecting abuse of the *flow* itself (purchase velocity, account-creation patterns), not just capping raw request counts.
  - q: What makes Improper Inventory Management a security risk rather than just operational untidiness?
    a: A forgotten `v1` endpoint or an internal-only route that leaked into the public gateway was never updated when the rest of the API got new auth checks, rate limits, or input validation, so it sits there as a fully functional bypass of every improvement made since it was abandoned — attackers specifically scan for exactly this kind of stale, unmonitored surface.
deeper:
  - label: "OWASP API Security Top 10 Course – Secure Your Web Apps"
    url: "https://www.youtube.com/watch?v=YYe0FdfdgDU"
related_formulas: []
---

## A different top ten

OWASP maintains two separate lists that share a name and a ten-item format and nothing else. `software-architect`'s `owasp-top-10-for-architects` covers the general web-application list — injection, cryptographic failures, security misconfiguration, the risks that apply to any server rendering HTML or handling a form post. The **OWASP API Security Top 10** is a narrower, API-specific list, rebuilt from incident data specifically about REST, GraphQL, and RPC-style APIs, and its emphasis is genuinely different. The general list's top entry is broken access control in the abstract; the API list's top entry is one very specific, very common shape of that failure — and understanding why it gets its own name is the fastest way into the rest of the list.

## BOLA: the vulnerability that explains the whole list

**Broken Object Level Authorization** happens when an endpoint correctly checks that a request is authenticated, and then never checks that the specific object being requested belongs to the requester. `GET /orders/123` returns order 123 to whoever asks, as long as they're logged in as *someone* — not necessarily the someone who placed that order. Change the ID to 124, 125, 126, and you're reading every customer's order history. This is the single most common real-world API vulnerability because it's the easiest mistake to make: the authentication middleware runs, a `200 OK` with real data comes back, and everything looks like it's working. The fix sounds trivial — `WHERE order_id = ? AND customer_id = ?` instead of `WHERE order_id = ?` — but it has to be applied at *every* endpoint that takes an object identifier, which is why it keeps recurring: one missed endpoint out of two hundred is enough.

## BFLA and BOPLA: the other two authorization failures

**Broken Function Level Authorization** is the vertical version of the same mistake — not "whose order is this" but "should this user be hitting this endpoint at all." A `DELETE /users/{id}` or `POST /admin/refund` reachable by a regular authenticated user because the admin check lives only in the frontend's navigation, not in the API, is BFLA. `authorization-models-rbac-abac-and-beyond`, which this lesson assumes, is exactly the toolkit for closing this one correctly — the point here is that having a role model doesn't help if it isn't enforced on every route.

**Broken Object Property Level Authorization** is the field-level version, and it's a 2023 merge of two 2019 categories: excessive data exposure (a response includes fields the client shouldn't see, trusting the frontend to hide them) and mass assignment (a request body is deserialized straight into a model, so a client can set fields — `role`, `account_balance`, `is_verified` — that were never meant to be client-writable). Both failures share the same root cause: the API's serialization and deserialization logic mirrors the database schema instead of an explicit, deliberately scoped contract.

## Resource exhaustion and business-logic abuse

**Unrestricted Resource Consumption** is what happens when nothing bounds how expensive a request can be — no cap on page size (`?limit=5000000`), no limit on recursive GraphQL query depth, no cost accounting at all — so a single legitimate-looking request can exhaust CPU, memory, or a downstream dependency. **Unrestricted Access to Sensitive Business Flows** is the same idea one layer up: no individual request is malformed, but nothing stops automated abuse of a flow that matters to the business specifically — buying out a limited product drop in milliseconds, farming referral bonuses with scripted signups, scraping a whole pricing catalog. Rate limiting (covered at the infrastructure level in `system-design`) caps raw request volume; this category is about detecting abuse of a *flow*, which usually needs velocity and pattern checks a generic rate limiter doesn't do.

## Trusting the network and the inventory

**Server-Side Request Forgery** shows up wherever an API fetches a URL on a caller's behalf — a webhook registration, an image-proxy endpoint, an "import from URL" feature — and a malicious caller points it at an internal address instead, like a cloud metadata endpoint, turning a convenience feature into a path to infrastructure the public internet was never meant to reach. **Security Misconfiguration** overlaps with the general OWASP list but has API-specific defaults worth naming: verbose stack traces in error responses, permissive CORS wildcards, an API gateway with default credentials still active. **Improper Inventory Management** is the risk of the API you forgot you're running: an old `v1` left live after `api-versioning-strategies`' sunset process was skipped, a staging deployment pointed at production data, an internal debug route that shipped to the public gateway by accident. None of the other nine items protect an endpoint your security review never knew existed.

## Unsafe consumption of APIs

The last category inverts the usual threat model: your API can be the victim, not the attacker's entry point, if it blindly trusts a third-party API's response — a webhook payload, an upstream partner's data feed — the same way it should never blindly trust client input. Deserializing and acting on unvalidated third-party data is how an SSRF-style issue or an injection vulnerability sneaks in through a dependency you don't control instead of through your own public surface.

Together these ten categories are the concrete, day-to-day version of what `api-performance-metrics-and-monitoring` and `data-privacy-and-compliance-standards-for-apis`, the next two lessons in this group, build on: you can't monitor or legally defend an API contract you haven't first made resistant to having its objects, functions, fields, and business flows walked straight through by anyone who's merely authenticated, not authorized.
