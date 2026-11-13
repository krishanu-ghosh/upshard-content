---
title: "Testing Fundamentals: The Test Pyramid"
description: Unit, integration, and end-to-end tests, what makes a unit test good, and why mocking exists to isolate the thing under test.
roadmap: computer-science
node: testing-fundamentals
group: concurrency-and-practices
video:
  provider: youtube
  id: "u6QfIXgjwGQ"
  start: 0
  end: null
creator:
  name: Fireship
  channel_url: https://www.youtube.com/@Fireship
duration_seconds: 136
level: intermediate
prerequisites:
  - version-control-internals
takeaways:
  - "The test pyramid describes a ratio, not just a list: many fast unit tests at the base, fewer integration tests in the middle, and a small number of end-to-end tests at the top."
  - "A unit test checks one small piece of code in isolation — typically a single function or class — with every dependency it touches replaced or controlled."
  - "A good unit test is fast, isolated (doesn't depend on other tests or external state), and deterministic — it gives the same result every single time it runs, in any order."
  - "An integration test checks that two or more real components work correctly together, like a service and its actual database, which a unit test by design can't verify."
  - "An end-to-end test drives the whole system the way a real user would, which catches real bugs but makes it the slowest and most fragile layer — which is exactly why the pyramid keeps that layer thin."
  - "Mocking and stubbing exist to isolate the thing under test from its dependencies, trading some realism for speed, determinism, and the ability to easily simulate error conditions that are hard to trigger for real."
check_questions:
  - q: "Why would a team want hundreds of unit tests but only a handful of end-to-end tests, instead of the other way around?"
    a: "Unit tests run in milliseconds with no external dependencies, so having hundreds of them costs almost nothing and gives fast, precise feedback about exactly which function broke, while end-to-end tests are slow and brittle — they need a running system, real network calls, and a UI — so keeping only a few of the most critical user flows at that layer keeps the whole suite fast enough to run constantly."
  - q: "A unit test for a function sometimes passes and sometimes fails with no code changes in between. What property of a good unit test is being violated, and why does that make the test nearly useless?"
    a: "It isn't deterministic — if a test's outcome depends on something uncontrolled, like real time, random numbers, network state, or test execution order, a failure stops telling you anything reliable about whether the code is broken, and a team quickly learns to ignore 'flaky' failures, which defeats the entire point of having the test."
  - q: "Why mock a database call in a unit test for a function that processes an order, instead of just running the test against a real test database?"
    a: "A real database makes the test slow, dependent on external state that has to be set up and torn down, and hard to use for simulating conditions like a connection timeout or a constraint violation — mocking the database call lets the test focus purely on the order-processing logic, run in milliseconds, and simulate any dependency behavior on demand, while a real database is exactly what an integration test is for instead."
deeper:
  - label: "TestPyramid (Martin Fowler)"
    url: "https://martinfowler.com/bliki/TestPyramid.html"
related_formulas: []
---

## Why one kind of test was never going to be enough

Every automated test answers the same underlying question — "does the code still do what it's supposed to?" — but no single kind of test can answer it cheaply, quickly, and realistically all at once. The **test pyramid**, a model popularized by Mike Cohn, organizes tests into layers that trade off those three qualities differently, and gives a rough guide for how many of each to write.

## The three layers

**Unit tests**, at the base, test one small unit of code — usually a single function or method — in complete isolation from everything around it. If the function being tested calls out to a database, a network API, or another class with complex behavior, a unit test replaces those dependencies with controlled stand-ins (more on this below) so that only the logic inside the function itself is actually being exercised. Because there's no real I/O, no real network, and no real external system involved, unit tests run in milliseconds and can number in the thousands without slowing anyone down.

**Integration tests**, in the middle, deliberately bring real components together and check that they cooperate correctly — a service talking to an actual (test) database, two internal modules calling each other for real, or your code talking to a real third-party API in a sandbox mode. This is the layer that catches the class of bug a unit test structurally cannot: the SQL query that's syntactically fine but wrong against the real schema, the API contract that doesn't match what the other service actually returns, the serialization format that round-trips differently than expected. Integration tests are slower than unit tests — they need real processes, real connections, sometimes real network latency — so there are usually far fewer of them.

**End-to-end (E2E) tests**, at the top, drive the entire assembled system the way an actual user would: launching a browser, clicking through a real UI, submitting a real form, and checking that the right thing eventually happens. This is the only layer that verifies the *whole* system actually works together, start to finish, but it's also the slowest (seconds to minutes per test) and the most fragile — a UI tweak, a slow network, or a timing issue unrelated to the actual feature can make an E2E test fail for reasons that have nothing to do with a real bug. That fragility is precisely why the pyramid keeps this layer thin: a handful of E2E tests covering the most critical user journeys (sign up, check out, log in) catches the bugs only full integration reveals, without making the whole suite slow or flaky.

## What makes a unit test actually good

Writing a function and a test that exercises it isn't enough to call it a good unit test. Three properties matter:

- **Fast.** A unit test should run in milliseconds. If a developer has to wait seconds for feedback, they stop running tests before committing, which defeats the purpose of having them.
- **Isolated.** A unit test shouldn't depend on other tests having run first, shouldn't depend on the order tests execute in, and shouldn't depend on any state left over from a previous run (a file on disk, a row in a shared database, a global variable another test mutated). Each test should be able to run completely alone and get the same result.
- **Deterministic.** Given the same code, a unit test must produce the same result every single time — pass or fail, with no in-between. A test that sometimes fails for no code-related reason (a "flaky" test) is usually hiding one of: real wall-clock time, unseeded randomness, network calls, or reliance on execution order. Flaky tests are worse than no tests at all, because teams learn to re-run them until green instead of trusting them, which quietly erodes the value of the entire suite.

## Why mocking and stubbing exist

A **mock** (or **stub** — the terms are often used loosely, though a stub just returns canned data while a mock additionally verifies it was called correctly) is a stand-in object that replaces a real dependency during a test. If the function under test calls `chargeCreditCard()`, a unit test doesn't want to actually charge a credit card — it wants to verify that *its own logic* calls `chargeCreditCard()` with the right amount when it should, and handles the response (success or failure) correctly either way.

Mocking exists to solve exactly the problem the three properties above create: a real dependency is often slow (a network call), nondeterministic (today's date, a random ID), hard to control (triggering a database timeout on demand), or simply unavailable in a test environment (a third-party payment API you don't want to actually call). Replacing it with a mock that returns exactly what you tell it to — including conditions that are difficult or dangerous to trigger for real, like "the payment API is down" — lets a unit test stay fast, isolated, and deterministic while still proving the code under test handles every case correctly.

The trade-off is realism: a mock only behaves the way you told it to, so a test built entirely on mocks can pass while the real integration is subtly broken. That's not a flaw in unit testing — it's exactly why the pyramid has integration and end-to-end layers above it. Unit tests, backed by mocks, prove the logic is correct in isolation; the layers above prove the pieces actually fit together in reality. Neither layer is redundant with the other, which is the whole reason the pyramid has more than one layer in the first place.
