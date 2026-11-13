---
title: "Contract Testing for APIs"
description: Consumer-driven contract testing against the OpenAPI spec — catching a breaking API change before either side of the boundary relies on it.
roadmap: api-design
node: contract-testing-for-apis
group: documentation-testing-and-quality
video:
  provider: youtube
  id: "IetyhDr48RI"
  start: 0
  end: null
creator:
  name: PactFlow
  channel_url: https://www.youtube.com/@PactFlow
duration_seconds: 629
level: intermediate
prerequisites:
  - openapi-and-api-documentation-tooling
takeaways:
  - "Contract testing targets one specific failure mode: a provider team changes their API in a way that breaks a consumer, discovered only when it hits production, because nobody's test suite actually exercises the real boundary between the two services without standing up both of them together."
  - "In consumer-driven contract testing (the Pact pattern), the consumer writes a test against a mock of the provider; that run generates a contract file recording exactly what the consumer expects — specific requests and the response shapes it depends on, not the provider's entire API surface."
  - "The provider then replays every consumer's contract against its real implementation in CI, as a verification step — if any expectation fails, the provider's pipeline fails before the breaking change ships, not after a consumer's production traffic does."
  - "Contract testing beats end-to-end integration testing at this specific boundary because it needs only the provider (or consumer) under test plus the other side's contract file — not a full environment with every dependent service running, which is what makes it fast enough to run on every commit instead of nightly."
  - "Schema validation testing is a different, complementary check: validating real request/response traffic against the committed OpenAPI spec (with a tool like Dredd or Schemathesis) catches \"the spec says X, the server actually returns Y\" even when no consumer's specific contract happens to exercise that field — contract tests check what a consumer actually relies on, schema validation checks the spec is telling the truth at all."
  - "This isn't the unit/integration/e2e pyramid from `computer-science`'s `testing-fundamentals` — contract testing is a layer added specifically at the API boundary between independently deployed services, orthogonal to how either side tests its own internals."
check_questions:
  - q: Who writes the contract in consumer-driven contract testing, and why does that matter?
    a: The consumer does, by writing a test against a mock of the provider that captures exactly what that consumer actually calls and depends on — not the provider's full API surface. That matters because it means the provider only has to keep the parts of its API that consumers genuinely use working, and the contract stays scoped to real usage instead of growing into a second, redundant copy of the whole spec.
  - q: Why is contract testing faster than standing up a full end-to-end integration test for the same two services?
    a: A contract test only needs the one service under test plus the other side's contract file (a mock built from the consumer's recorded expectations, or the provider verifying against it directly) — no second real service, no shared database, no network between two live deployments. An end-to-end test needs every dependent service actually running, which is slow to set up, flaky under real network conditions, and gets dramatically slower as the number of services a given flow touches grows.
  - q: A team has contract tests passing between a consumer and a provider, but a bug ships where the provider's real response doesn't match what its own OpenAPI spec documents for a field the consumer's contract never touched. What test would have caught that, and why didn't contract testing?
    a: Schema validation testing against the OpenAPI spec itself (e.g. with Schemathesis or Dredd) would have caught it, because it checks the provider's real responses against everything the spec claims, not just the subset a given consumer's contract happens to exercise. Contract testing is scoped to what consumers actually rely on, which is its strength for catching real breaking changes fast, but it means a field no contract touches can still be wrong relative to the spec without any contract test ever noticing.
deeper:
  - label: "Introduction to contract testing, Part 1: The problem with end-to-end integrated tests"
    url: "https://www.youtube.com/watch?v=U05q0zJsKsU"
related_formulas: []
---

## The problem that lives specifically at the API boundary

`testing-fundamentals` in `computer-science` already covers why you want many fast unit tests and few slow end-to-end ones. This lesson assumes that and goes one layer up: the specific problem of verifying that two independently deployed services — a consumer calling an API and the provider serving it — stay compatible as both change over time, without either side needing the other one actually running to find out.

The naive answer is end-to-end integration tests: stand up the consumer, stand up the provider, point one at the other, and run a real request through. That works, but it scales badly the moment more than two or three services are involved. Every service a flow touches has to be deployed and healthy for the test to run at all, which makes the suite slow, flaky under real network and startup-order conditions, and expensive enough that teams push it to a nightly job instead of running it on every commit — exactly when you'd most want fast feedback on a breaking change. By the time that nightly job fails, five other commits have landed on top of the one that broke it.

## The Pact pattern: consumer-driven contracts

Consumer-driven contract testing flips the order of what gets verified against what. The **consumer** writes a test against a mock of the provider, asserting the specific requests it sends and the specific response shapes it depends on — nothing about the provider's API it doesn't actually use. Running that test produces a **contract file** (a Pact file, in Pact's terminology): a recorded, machine-readable artifact of exactly what this consumer expects from this provider.

The **provider** then takes that contract file and replays it in its own CI pipeline — a verification step that sends the exact requests the contract records at the provider's real implementation and checks the real responses against what the consumer expects. If a provider team ships a change that would have broken the consumer — renaming a field, changing a status code, dropping a header the consumer reads — the verification step fails in the provider's own pipeline, before the change merges, not after it reaches the consumer's production traffic. A Pact Broker (or a hosted equivalent like PactFlow) sits in the middle, holding the latest contracts from every consumer so a provider's CI can pull and verify against all of them, and so a provider can answer "is it safe for me to deploy this" by checking whether every consumer's contract still passes.

This is why contract testing is faster than full end-to-end testing at this specific boundary: a provider verifying a contract needs only itself and a file, not a live consumer service running somewhere; a consumer testing against its own contract needs only a mock built from that same file, not a live provider. Neither side needs the other's full dependency graph running, which is exactly what makes this fast enough to run on every pull request instead of being quarantined to an overnight job.

## Schema validation: the check contract testing doesn't do

Contract testing is deliberately scoped to what a consumer actually relies on — that's its strength, not a gap, since it keeps the contract small and focused on real usage instead of growing into a second, parallel copy of the entire API. But that scoping means it won't catch a field, status code, or response shape that no consumer's contract happens to touch drifting away from what the committed OpenAPI spec (from `openapi-and-api-documentation-tooling`) says it should be.

Schema validation testing closes that gap directly: a tool like Dredd or Schemathesis sends requests at the live API and validates the real responses against the OpenAPI document itself, field by field, independent of any particular consumer. Where contract testing answers "does this response still satisfy what my actual consumers depend on," schema validation answers "does this response still match what we told everyone, in the spec, it would be." Run both, and you've covered the two real ways an API boundary breaks: a response consumers depend on changing shape, and a response drifting away from its own published contract even where nobody's using the part that moved yet.

## A worked example

A billing service exposes `GET /invoices/{id}`, and a dashboard consumer reads `status`, `amountDue`, and `dueDate` from the response. The dashboard team's Pact test asserts exactly that shape and generates a contract. If the billing team later renames `amountDue` to `balanceDue` without checking contracts first, their CI's verification step replays the dashboard's contract against the real endpoint, gets a response missing the field the contract expects, and fails — caught in the billing team's own pipeline, with a clear pointer to which consumer broke and why, rather than as a support ticket after the dashboard silently starts rendering `undefined`. Meanwhile, if the same response also has a `currency` field the spec documents as a required ISO 4217 code but the real implementation sometimes omits, no contract catches that — the dashboard never reads `currency` — but a schema validation pass against the OpenAPI spec does, because it checks every documented field, not just the ones a consumer happened to depend on. `mocking-and-virtualizing-apis`, next in this group, is where that same OpenAPI spec gets reused again — this time to generate a working mock server instead of to validate real traffic against it.
