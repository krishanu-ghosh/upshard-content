---
title: "Why Code Review Matters"
description: What code review actually catches — shared ownership, consistency, and surfaced alternatives — and the failures it's realistically bad at finding.
roadmap: code-review
node: why-code-review-matters
group: code-review-practice
video:
  provider: youtube
  id: "uIwl01Nazdg"
  start: 0
  end: null
creator:
  name: EuroPython Conference
  channel_url: https://www.youtube.com/@EuroPythonConference
duration_seconds: 2657
level: beginner
prerequisites: []
takeaways:
  - "The strongest case for code review isn't bug-catching — it's that review is the one point in the whole process where a second person is guaranteed to look at the code before it ships, which makes it the natural place to spread knowledge of how a system actually works."
  - "A codebase where only one person understands a given subsystem has a bus factor of one for that subsystem, and review is one of the few habits that lowers it without anyone scheduling a dedicated knowledge-transfer meeting."
  - "Reviewers catch inconsistency with team convention more reliably than they catch logic errors, because spotting \"this doesn't match how we normally do this\" takes far less context than tracing a bug through unfamiliar code."
  - "A reviewer who has never touched the problem the author just solved often proposes a different, sometimes better, approach — not because they're smarter, but because they haven't spent the last three hours anchored to the first solution that worked."
  - "Review is a weak tool for catching deep architectural problems after the fact, because by the time a pull request exists the big decisions — what talks to what, where the boundaries are — are usually already locked in by the code that surrounds it."
  - "Treating review as a substitute for tests just moves the bug-finding work from an automated, repeatable check to a human skimming a diff once, which is a strictly worse deal."
check_questions:
  - q: Why does the knowledge-transfer argument for code review matter more than the bug-catching argument?
    a: Catching bugs is a function of how carefully a reviewer reads, which is inconsistent — but every review is a guaranteed moment where someone other than the author looks at the change, which reliably spreads understanding of the system even on the days the reviewer misses a bug.
  - q: What does "bus factor" mean, and how does routine review lower it?
    a: Bus factor is the number of people who'd need to disappear before a piece of knowledge is lost to the team entirely — routine review lowers it because every reviewed change leaves at least one more person who has seen that part of the code and could plausibly maintain it later.
  - q: Why are reviewers often better at catching convention violations than at catching logic bugs?
    a: Spotting that a change doesn't match the team's usual patterns only requires knowing those patterns, while catching a logic bug requires reconstructing the author's intent and tracing execution through code the reviewer didn't write — the first is a much lower bar than the second.
  - q: What's a realistic limit on what code review can catch?
    a: Review struggles to catch deep architectural problems, because by the time a change reaches a pull request the big structural decisions are usually already baked into the surrounding code, and a reviewer looking at one diff rarely has the vantage point to question the foundation it sits on.
deeper:
  - label: "Code Review Skills for Pythonistas"
    url: "https://www.youtube.com/watch?v=6L3ZVLtSeo8"
related_formulas: []
---

## The case nobody argues from

Ask most engineers why code review exists and you get the same answer: it catches bugs before they ship. That's true, but it's not the strongest reason to do it, and leaning on it too hard is how review processes rot — once a team decides the point is bug-catching, and the tests pass, and nobody's found a bug in the last ten reviews, the whole ritual starts to feel like overhead. Dougal Matthews' EuroPython talk makes a different, more durable case: review earns its keep mostly as a knowledge-transfer mechanism, and only secondarily as a bug filter.

Here's the reasoning. A bug is found by review only if the reviewer happens to notice it, which depends on luck, familiarity with the code, and how carefully they're reading that day. Knowledge transfer isn't luck-dependent — it happens every single time, whether or not a bug turns up. The moment a reviewer opens a diff, they've learned something about a part of the system they may never have touched. That's a guaranteed return on a review, independent of whether the change itself was correct.

## The bus factor problem

Picture a team where one person wrote the billing reconciliation job two years ago, and nobody else has opened that file since. It works, so nobody has a reason to. Then that person goes on leave, or leaves the company, and a bug shows up in reconciliation — now someone has to learn the entire subsystem under time pressure, from a standing start, with the original author unreachable.

That's the bus factor: the number of people who'd have to vanish before a piece of knowledge disappears from the team. A bus factor of one isn't a hypothetical risk, it's the default state of any code nobody else has read. Review is one of the few routine habits that pushes against it — every time someone reviews a change to that reconciliation job, they've now seen how it works, even if they don't remember every detail. It doesn't need to be a dedicated knowledge-transfer session to count; it happens as a side effect of doing the review at all.

## Keeping the team speaking one dialect

Review is also unusually good at catching something that has nothing to do with correctness: drift from team convention. If everyone on a team writes error handling one way, and a change introduces a different pattern, a reviewer will notice — not because they traced the logic and found a flaw, but because the shape of the code doesn't match what they're used to seeing. That's a much lower bar than finding a genuine bug. You don't need to understand what a function does to notice it doesn't look like the rest of the codebase.

This matters because consistency has compounding value. A codebase where every module solves the same kind of problem in its own idiosyncratic way is a codebase where every new feature starts with "wait, how do we do this here?" Review, cheaply and as a side effect, keeps that question rare.

## A second opinion you didn't ask for

There's one more thing review supplies that's easy to undervalue: a reviewer who hasn't spent the last few hours staring at the problem. An author who's been deep in a bug fix is anchored to the first approach that worked — they stop looking once something passes. A reviewer coming in fresh sometimes proposes a cleaner alternative, not from superior skill, just from not being anchored yet. That's a real benefit, and it's one you only get from a second person looking, which is exactly what review forces to happen.

## What review can't do

None of this makes review a cure-all. It's a weak tool for catching deep architectural problems, because those decisions are usually locked in long before a pull request exists — by the time you're reviewing a diff, the shape of the system around it is already fixed. It's not a substitute for tests either: a human skimming a diff once is a worse bug-finding mechanism than an automated check that runs on every commit forever. And it doesn't scale to equal scrutiny everywhere — a one-line typo fix and a change to the payment pipeline don't deserve the same five minutes, and pretending otherwise is how review becomes theater instead of a genuine second look. Knowing what review is realistically good for is what keeps it useful instead of ceremonial.
