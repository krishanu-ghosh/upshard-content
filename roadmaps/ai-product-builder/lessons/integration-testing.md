---
title: Integration Testing
description: Testing how the frontend, API, and database actually agree with each other, which is where AI-generated pieces most often disagree.
roadmap: ai-product-builder
node: integration-testing
group: testing-and-quality
video:
  provider: youtube
  id: "NLjiTMJDIcY"
  start: 0
  end: null
creator:
  name: Code Made Simple
  channel_url: https://www.youtube.com/@codemadesimple1043
duration_seconds: 513
level: intermediate
prerequisites:
  - unit-testing-ai-generated-code
takeaways:
  - Unit tests confirm that one function behaves correctly in isolation; integration tests confirm that the pieces around it — a route handler, a database call, a response shape — actually agree once they're wired together.
  - "A request-response cycle through a real route is what an integration test exercises, which catches problems that only exist at the seam: wrong status codes, missing fields, a database query that doesn't match the schema it's run against."
  - Tools like Supertest let you send real HTTP requests at your Express (or similar) app in a test, without starting a separate server process, and assert on the actual response.
  - AI-generated apps are especially prone to seam bugs, because the frontend, the API, and the database layer may each have been generated in a separate prompt, each making its own locally-reasonable assumption about what the others expect.
  - Integration tests should run against something close to a real backend — a test database, a running API — rather than mocks of every dependency, or you've just tested that the mocks agree with each other.
check_questions:
  - q: What does an integration test check that a unit test, by design, cannot?
    a: It checks that separate pieces actually work correctly together — that an API route calls the database correctly, returns the status code and shape the frontend expects, and so on — rather than checking any one function in isolation from everything around it.
  - q: Why does a tool like Supertest let you test an Express app without running a separate server?
    a: It sends simulated HTTP requests directly into the app's request-handling pipeline in-process and captures the real response, giving you the same request-response cycle a live server would produce without the overhead of actually binding a port.
  - q: Why are AI-generated apps especially likely to have bugs that only integration testing catches?
    a: The frontend, API, and database layers may each have been generated in separate prompts or sessions, so each piece can be internally consistent and still make a different, locally reasonable assumption about field names, response shapes, or error formats than the piece next to it.
deeper:
  - label: "API Testing with Supertest: A Beginner's Guide"
    url: https://www.youtube.com/watch?v=hIRY89S-HLA
related_formulas: []
---

## The bug that lives between the pieces

A unit test tells you that a single function does what it's supposed to, given a specific input. It tells you nothing about whether that function is ever called correctly by the thing that calls it, or whether what it returns is actually usable by the thing that receives it. That gap — the seam between pieces — is where a whole class of real bugs lives: a frontend that expects a field called `userId` talking to an API that returns `user_id`; a route that returns a 200 with an error message in the body instead of an actual error status; a database query that works against the schema in your head but not the schema that's actually deployed. Integration testing is built specifically to catch that class of bug, by testing the seam itself rather than either side of it alone.

## What an integration test actually exercises

An integration test for a web app typically drives a real request through a real route: it hits an endpoint the way a client would, lets the route handler do its actual work — parsing the request, querying or writing to a database, building a response — and then asserts on what comes back. The video behind this lesson shows exactly this pattern using Supertest against Express controllers: a test sends a request into the app directly, in-process, without needing to bind a port or run a separate server, and gets back the real response the app would have produced for a live client. That's the core move of integration testing — you're not simulating the route's logic, you're running it, with only the outermost edges (a real client, a real server process) left out.

## Why generated apps fail here more than code you wrote end to end

If you build an app yourself from the frontend through the API to the database, you tend to keep the contract between layers consistent in your head as you go — you remember what you called a field three files ago. AI-generated apps don't come with that guarantee. It's common to prototype a frontend in one session, generate an API in another, and wire up a database schema in a third, each pass making its own reasonable-sounding decision about naming, shape, or error handling. Each piece can pass its own unit tests perfectly and still not agree with its neighbors. Integration testing is the layer that actually checks the thing you can't verify by reading any single file: does what the frontend sends match what the API expects, and does what the API returns match what the frontend can use.

## Testing against something real, not a tower of mocks

It's tempting to make integration tests fast and isolated by mocking the database, mocking the API, mocking everything except the one function under test — but that quietly turns an integration test back into a unit test wearing a disguise, and it stops catching the exact bugs this layer exists to catch. A meaningful integration test runs against something close to the real thing: a test database with a real schema, a real instance of the API. It costs a bit more setup and runs a bit slower than a unit test, and that's the right trade — you're buying confidence that the actual seams hold, not that your assumptions about the seams agree with themselves.

## Where this sits in the testing stack

Integration tests don't replace unit tests, and they don't yet test the full user experience either — that's the next layer, end-to-end testing. What they add is confidence at exactly the boundary where independently generated pieces of an AI-built app are most likely to silently disagree, which makes this layer disproportionately valuable for the kind of apps this roadmap is about building.
