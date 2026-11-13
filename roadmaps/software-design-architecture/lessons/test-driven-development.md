---
title: "Test-Driven Development"
description: The red-green-refactor cycle, and how writing the test before the code changes the shape of the design you end up with.
roadmap: software-design-architecture
node: test-driven-development
group: code-quality-in-practice
video:
  provider: youtube
  id: "B1j6k2j2eJg"
  start: 0
  end: null
creator:
  name: ArjanCodes
  channel_url: https://www.youtube.com/@ArjanCodes
duration_seconds: 910
level: beginner
prerequisites:
  - code-smells-and-refactoring
takeaways:
  - "Test-driven development is a design discipline wearing a testing hat: the test gets written before the code it tests, which forces you to decide the shape of a function's interface before you've committed to how it works inside."
  - "Red means writing a test for behavior that doesn't exist yet and watching it fail — and the failure has to be for the right reason (the feature is missing), not a typo or a broken import, or the test was never actually testing anything."
  - "Green means writing the smallest amount of code that makes the failing test pass, even a deliberately fake or hardcoded answer — the goal of this step is only to get back to a passing state, not to write the final implementation."
  - "Refactor means improving the structure of code that's already proven correct by the test, with the test itself as the net that catches any accidental change in behavior."
  - "Writing the test first forces a function to be callable in isolation before it's written, which is naturally hostile to functions that secretly depend on three other objects being constructed first — the discipline pushes toward small, decoupled units almost as a side effect."
  - "TDD's payoff is in the design pressure it applies during the cycle, not just in the regression suite left behind afterward — a codebase built test-first tends to end up more decomposed than the same codebase built with tests bolted on at the end."
check_questions:
  - q: Why does the Green step call for the smallest possible amount of code, even an obviously fake or hardcoded value, rather than the real implementation right away?
    a: The point of Green is only to confirm the test and the code agree on what "passing" looks like, as fast as possible — writing the full implementation immediately risks spending effort on logic the next test might reshape anyway, and conflates two different jobs (making it pass, then making it right) that are easier to do one at a time.
  - q: How does writing a test before the function exists change the design of that function, compared to writing the test afterward?
    a: A test written first has to call the function the way any other caller would, with no inside knowledge of its implementation, which forces the function's inputs and outputs to be simple and self-contained before a single line of its body exists. A test bolted on afterward tends to just confirm whatever shape the implementation already took — including tangled dependencies that are now awkward to test, which is usually solved by reaching for a mock instead of fixing the design.
  - q: A developer writes a failing test, then changes the test itself until it passes, without touching the real code. What part of the cycle did they skip, and why does that defeat the purpose of TDD?
    a: They skipped Green as it's supposed to work — passing is supposed to come from changing the actual implementation to satisfy a fixed, meaningful test, not from loosening the test until it accepts whatever the code already does. A test that gets weakened to match reality stops verifying anything, which quietly turns the whole red-green-refactor cycle into theater.
deeper:
  - label: "Does TDD Lead to Better Software Design? (Modern Software Engineering)"
    url: "https://www.youtube.com/watch?v=fSvQNG7Rz-8"
  - label: "Test Driven Development - What? Why? And How? (Modern Software Engineering)"
    url: "https://www.youtube.com/watch?v=llaUBH5oayw"
related_formulas: []
---

## Writing the test first is the whole point

It's tempting to think TDD is just "write tests, but early" — a scheduling choice. It isn't. This lesson assumes you already know what a unit test is and what the test pyramid looks like (covered in `computer-science`'s `testing-fundamentals`); what's different here is the order of operations, and the order is the entire idea. Writing the test before the code means you have to decide how a function will be *called* — its name, its inputs, its return value — before you've written a single line of how it works inside. That constraint, applied over and over across a whole codebase, is what shapes the design. It isn't a side benefit of TDD; it's the mechanism.

## The cycle: Red, Green, Refactor

**Red.** Write a test for behavior that doesn't exist yet, and run it. It has to fail — and it has to fail for the right reason. A test that fails because of a typo in the test file, or an import error, hasn't actually tested anything yet; watching it fail *because the feature is missing* is what confirms the test is wired up correctly.

**Green.** Write the smallest amount of code that makes the test pass. "Smallest" is deliberate — it's fine, even encouraged early on, to return a hardcoded value if that's literally what makes the test pass. The job of this step is narrowly defined: get back to a passing state, nothing more. Resisting the urge to write the "real" implementation here keeps the two concerns — correctness and design — from getting tangled into one step.

**Refactor.** Now, and only now, improve the structure of the code: rename something unclear, pull out a repeated fragment, simplify a condition. The test suite is what makes this safe — any change in behavior shows up as a failing test immediately, which is the entire reason refactoring (the previous lesson in this group) depends on a test suite existing in the first place.

Then the cycle repeats, one small test at a time, rather than one developer trying to design the whole module up front.

## Why the order creates small, decoupled units

Consider what happens when a test has to call a function with no knowledge of what's inside it. If that function secretly needs a live database connection, three other objects constructed in a specific order, and a global variable set just right, writing the test first turns that awkwardness into an immediate, visible wall — you hit it before the function is even implemented, not months later when someone tries to reuse it. The natural response is to shrink the function's dependencies until it's callable on its own. That's not a rule TDD states explicitly; it's design pressure that falls out of writing the test first. Testing the same code after it's written doesn't apply this pressure nearly as hard — by then the dependencies already exist, and it's easier to mock them away than to go back and decouple the design.

## Where the discipline breaks down

TDD fails quietly in a couple of predictable ways. One is writing the test and the implementation at the same time, which looks like TDD but skips the part where the test has to fail first — skip Red, and you've lost the one step that proves the test was capable of catching a real bug. The other is weakening a test until it matches whatever the code already does, instead of changing the code to satisfy a meaningful test — a test that bends to reality instead of constraining it has stopped doing its job, even though the suite still shows green.

## What this buys you, beyond the tests themselves

A team that adopts TDD gets a regression suite, same as any team that writes tests — but the suite is really a byproduct. The actual return is a codebase shaped, function by function, by the constraint of being callable before it existed. That's a different outcome than writing comprehensive tests after the fact for code that was designed without that pressure; the tests end up just as thorough, but the design underneath them tends to be more tangled, because nothing forced it to be small along the way.
