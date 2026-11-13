---
title: Production patterns
description: Versioning prompts like code, guardrails before a response ships, monitoring for drift, and feeding user feedback back into the eval set.
roadmap: prompt-engineering
node: production-patterns
group: optimization
video:
  provider: youtube
  id: "LzeC1AQ-U5o"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1043
level: intermediate
prerequisites: []
takeaways:
  - A prompt in production needs the same version control, rollback, and change tracking as any other code that affects user-facing behavior.
  - Guardrails validate a model's output before it reaches a user, catching format or safety failures that a prompt alone can't guarantee against.
  - Real user feedback — corrections, thumbs down, flagged outputs — is the highest-signal input for growing an eval set and deciding what to fix next.
check_questions:
  - q: Why should a prompt be versioned the same way application code is, instead of edited in place?
    a: A prompt is a dependency the rest of the application behaves differently around; editing it in place with no history makes a regression impossible to diagnose or roll back, exactly like shipping an unreviewed, untracked code change.
  - q: What's the difference between an eval running before deployment and a guardrail running in production?
    a: An eval checks a candidate prompt or model against a fixed test set before anything ships, to decide whether a change is an improvement. A guardrail checks each individual live output, after generation and before the user sees it, to catch failures an eval's aggregate score wouldn't block.
deeper: []
related_formulas: []
---

## Everything in this roadmap, now running in production

Every earlier lesson here — prompt anatomy, few-shot examples, structured output,
evaluation — assumed a prompt being designed and tested. This lesson is about what
changes once that prompt is live, serving real traffic, with real consequences when it
fails. The video behind this lesson, a 2023 AI Engineer Summit talk, frames the shift
around four building blocks: evaluations, retrieval, guardrails, and user feedback. The
specific tools it names have moved on since, but the shape of the problem — a prompt
alone is not a product — hasn't.

## Prompts are a dependency, so version them like one

A prompt embedded as a string somewhere in application code, edited in place whenever
someone wants to try something, is the single most common production mistake in this
space. It means no history of what changed, no way to tell which version produced a
given user's bad output, and no clean way to roll back a regression. Treat prompt text
the way you'd treat any other code that materially changes behavior: in version control,
changed through a reviewed process, tagged with what changed and why, with the ability
to point a specific deployment at a specific known-good version. The
[evaluation and testing](/roadmaps/prompt-engineering/evaluation-and-testing) lesson's
eval set is what tells you whether a new version is actually better before it replaces
the old one — a version bump without re-running the evals is just hoping.

## Guardrails: checking the output, not just the instructions

A prompt can increase the odds of a well-formed, safe, on-topic response, but it cannot
guarantee one — models still occasionally ignore formatting instructions, hallucinate,
or produce something that shouldn't reach a user. A guardrail is a check that runs after
generation and before display: validating that a structured response actually parses
against its schema (see [structured output formats](/roadmaps/prompt-engineering/structured-output-formats)),
scanning for disallowed content, or running a cheaper secondary model as a classifier to
flag suspicious outputs for review or automatic retry. The distinction from an eval
matters: an eval scores a prompt's aggregate quality on a fixed test set before you
ship a change; a guardrail inspects every single live output, one at a time, catching
failures an aggregate score could still let through.

## Monitoring for drift, not just outages

A prompt that worked well at launch can quietly degrade without ever producing an
error — the underlying model gets updated by its provider, user inputs shift as traffic
grows into cases nobody tested, or a prompt interacts badly with a new edge case that
simply wasn't common before. None of this shows up as a crash; it shows up as a slow
decline in output quality that nobody notices without deliberately tracking it. Treating
prompt performance as something to monitor continuously — spot-checking a sample of
real outputs on a schedule, tracking guardrail trigger rates over time, watching for
shifts in input patterns — is what catches this kind of silent regression before users
start to notice it themselves.

## Closing the loop with real feedback

The examples a team thinks to test before launch are never the full space of what
actual users send. Real feedback — explicit signals like thumbs up/down, and implicit
ones like a user immediately rephrasing their question because the first answer missed
— is the highest-value source of new test cases, because every one of them is a real
failure or success mode nobody invented in a planning meeting. The practical move is
mechanical: route flagged or corrected outputs back into the eval set as new test
cases, re-run the evals, and let that evidence — not a hunch about what might be
wrong — decide what the next prompt revision should actually change. A production
prompt isn't a thing you finish; it's a thing you keep closing this loop on.
