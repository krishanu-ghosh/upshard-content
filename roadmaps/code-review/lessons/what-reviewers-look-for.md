---
title: "What Reviewers Look For"
description: A priority order for code review — design and correctness first, then tests and readability, with style left to the linter.
roadmap: code-review
node: what-reviewers-look-for
group: code-review-practice
video:
  provider: youtube
  id: "EjwD7Pi7J_0"
  start: 0
  end: null
creator:
  name: JetBrains
  channel_url: https://www.youtube.com/@JetBrainsTV
duration_seconds: 730
level: beginner
prerequisites:
  - writing-a-reviewable-pull-request
takeaways:
  - "A review's attention budget is finite, so the order you spend it in matters as much as what you catch — design and correctness come before naming and formatting, because a beautifully styled solution to the wrong problem is still the wrong problem."
  - "A PR can be clean, short, and well-named while silently dropping an error or mishandling an edge case — readability makes code easy to review, it doesn't make the logic correct."
  - "Test coverage numbers don't tell a reviewer whether the tests exercise the behavior that actually matters — a test suite can hit every line and still never assert on the one edge case most likely to break in production."
  - "Anything a linter or formatter would flag automatically (spacing, import order, brace style) is not worth a human comment — raising it in review spends attention that should go toward design and correctness."
check_questions:
  - q: Why should a reviewer look at design and approach before checking variable names or formatting?
    a: Attention spent during a review is limited, and style issues are cheap to fix later while a wrong approach often means rewriting the change entirely — catching the wrong approach early saves far more work than catching a bad variable name early.
  - q: Why isn't "the tests pass" the same as "the tests are good"?
    a: Passing tests only confirm that the code does what the tests check for, not that the tests check for the behavior that matters — a suite that never exercises the actual edge case in question will pass right up until that edge case happens in production.
  - q: Why shouldn't a reviewer leave comments about things a linter already catches?
    a: A linter runs instantly and never gets tired, so it should own every mechanical rule a tool can enforce, leaving the reviewer's limited time and attention for the judgment calls — design, correctness, and clarity — that no tool can make.
deeper: []
related_formulas: []
---

## The order problem

Open ten pull requests and you'll find ten different instincts about where to start. Some reviewers start at the top of the diff and comment line by line, catching a typo here, a missing semicolon there. It feels productive — every comment is a correction. But it's the wrong place to spend the first ten minutes. If the change solves the wrong problem, or solves the right problem in a way that will need to be ripped out next quarter, every one of those line comments was wasted effort. Trisha Gee's talk on code review lays out something closer to a priority order: look at the shape of the solution before you look at the shape of the code.

That order isn't arbitrary. Attention during a review is a limited resource — most reviewers give a PR a fixed amount of focus before fatigue sets in and comments get shallower. Spend that budget on the things that are expensive to fix later. A design problem found on day one of a three-week feature costs a conversation. The same design problem found after merge costs a migration.

## Design and approach, first

Before reading a single line of implementation, ask: does this solve the problem it claims to solve, and is this *roughly* the right way to solve it? A PR that adds caching to a slow endpoint should be checked against the actual bottleneck — if the slowness was in a database query and the PR caches the API response instead, the change is tidy, tested, and wrong. This is not the place to debate whether a class should be split in two (that's a design-quality question for another lesson) — it's the place to check that the overall approach actually addresses the problem the ticket describes, and doesn't introduce a dependency or a coupling the team will regret.

## Correctness: does the logic actually do what it claims

Once the approach checks out, read the logic as if you're about to run it in your head with a hostile input. A function that calculates a discount should be checked against the empty cart, the cart with one item, the negative quantity nobody should send but someone eventually will. Here's the contrast worth holding onto: a PR that's visually clean — short functions, consistent naming — can still silently swallow an error, catching an exception and doing nothing with it so the caller never finds out something failed. A PR that's visually ugly — a sprawling conditional, inconsistent spacing — can still correctly handle every edge case the first one missed. Readability and correctness are different axes. A reviewer who only judges the first is approving bugs because they were well-formatted.

## Do the tests test the right thing

New tests in a PR answer a narrower question than "is this covered": do they assert on the behavior that would actually break if the logic above were wrong? A test that calls a function and checks it doesn't throw isn't testing much. A test that feeds in the boundary case — the zero, the empty list, the duplicate entry — and checks the specific output is testing something. Coverage percentage is a proxy, and proxies get gamed, even unintentionally: it's easy to write a test that executes a line without ever checking what that line produced. Ask what would have to go wrong in the code for this test to fail. If the answer is "nothing, really," the test isn't earning its place in the suite.

## Readability, and the security spots that need a second look

Only after design, correctness, and tests hold up does naming and structure earn attention — and it earns it because readability is itself a correctness tool: a reviewer, and every future maintainer, needs to be able to follow the logic without reconstructing the author's intent from scratch. A variable named `data` or a function that does three unrelated things slows down every future review of that code, including the next one.

Alongside readability, certain spots deserve a closer look regardless of where they fall in the diff: anywhere user input gets parsed or passed to a query, anywhere an authorization check could be skipped on one code path but not another, anywhere a secret or credential might end up logged or hardcoded. These aren't a separate pass — they're a heightened version of the correctness check, applied to the handful of places where a quiet mistake has an outsized cost.

## What a reviewer shouldn't spend time on

Indentation, import ordering, brace placement, trailing whitespace — if a linter or formatter can catch it, a human commenting on it is a waste of the same limited attention this lesson opened with. Worse, a PR full of style nitpicks trains authors to expect review as a formatting gate, not a check on whether the code actually works. Automating those checks away is its own lesson later in this roadmap, but the practical rule for a reviewer today is simple: if a tool could have said it, let the tool say it, and spend your comment on something a tool can't see.
