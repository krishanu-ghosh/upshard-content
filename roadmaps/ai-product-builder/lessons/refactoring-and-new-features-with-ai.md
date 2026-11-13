---
title: Refactoring and New Features with AI
description: Why structural refactors and new features need upfront research and a reviewed plan from an AI agent, unlike a small one-shot edit.
roadmap: ai-product-builder
node: refactoring-and-new-features-with-ai
group: building-with-ai-coding-agents
video:
  provider: youtube
  id: "rmvDxxNubIg"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1231
level: advanced
prerequisites:
  - prompting-for-code-changes
takeaways:
  - A structural refactor or a genuinely new feature touches enough of a codebase that a single one-shot prompt rarely produces a safe or coherent result — the task needs to be broken into stages.
  - "The research stage matters as much as the implementation stage: an agent needs to understand existing patterns, dependencies, and constraints before it proposes how to change them."
  - A written plan, reviewed before any code is touched, is where a wrong architectural direction gets caught cheaply — the same mistake found in the diff is far more expensive to unwind.
  - Large changes go better as a sequence of smaller, independently reviewable steps than as one enormous diff, even when the end goal is a single cohesive feature or refactor.
  - The discipline this calls for — research, plan, review, then implement — is the same discipline an experienced engineer applies to a hard problem; an agent doesn't remove the need for it, it just needs it stated explicitly.
check_questions:
  - q: Why does a large refactor fail more often with a single one-shot prompt than a small bug fix does?
    a: A large refactor touches many interdependent parts of a codebase at once, so without an understanding of existing patterns and constraints an agent is likely to make locally reasonable choices that don't cohere globally — the kind of mistake a tiny, isolated change has no room to make.
  - q: What's the purpose of having an agent produce a plan before writing any code for a big task?
    a: It surfaces the intended approach — which files, in what order, with what tradeoffs — while it's still cheap to redirect, so a wrong architectural choice gets caught before it's implemented across many files rather than after.
  - q: Why is breaking a large feature into smaller steps better than requesting it as one big change, even if the end result is meant to be one cohesive feature?
    a: Each smaller step produces a diff that can actually be reviewed and verified on its own, and a mistake in one step is easier to isolate and fix than the same mistake buried inside one sprawling, unreviewable change.
deeper:
  - label: "How to modernize legacy codebases using AI coding agents (Step by step guide)"
    url: "https://www.youtube.com/watch?v=RKvckuC6Bsk"
related_formulas: []
---

## Where small edits stop working

Everything in the previous lesson about scoping a request to something reviewable still holds — but a structural refactor spanning a dozen files, or a genuinely new feature with its own data model and UI, can't be scoped down to a single small prompt without losing the thing that makes it one cohesive change. Asking an agent to "extract this shared logic into a service and update every caller" or "add multi-tenant support" in one shot forces it to make a long sequence of interdependent decisions with no checkpoint in between. Any one of those decisions being wrong — a naming choice, a boundary drawn in the wrong place — propagates into everything built on top of it before anyone has a chance to notice.

## Research before anything gets written

The first stage of a large change isn't writing code at all — it's understanding what already exists. Before touching a shared module, an agent (or an engineer) needs to know who calls it, what assumptions those callers rely on, which parts are safe to change freely and which are load-bearing in ways that aren't obvious from the code alone. Skipping this stage is the single most common reason an AI-driven refactor goes wrong: the agent proposes a locally sensible change that breaks a caller three files away it never looked at. Explicitly asking an agent to investigate and summarize the relevant parts of the codebase before proposing anything is not extra ceremony — it's the step that makes everything after it trustworthy.

## A plan is a cheap place to fail

Once the agent understands the terrain, the next artifact should be a plan: the sequence of changes it intends to make, in what order, and why. This plan costs almost nothing to produce and almost nothing to review, which makes it the cheapest point in the whole process to catch a bad direction. If the plan proposes restructuring a module you'd rather leave alone, or introduces a dependency you don't want, redirecting at this stage takes a sentence. Catching the same issue after the agent has already implemented it across ten files means reviewing a much larger diff and then asking for a rewrite — the same correction, at a much higher cost, simply because it was caught too late.

## Implementing as a sequence, not a leap

Even after a plan is approved, the best way to execute a large refactor or feature is still as a series of smaller steps, each one a diff that can be reviewed and verified before the next begins. "Introduce the new interface, but don't switch callers yet" is a reviewable step. "Migrate the first three callers" is another. "Remove the old interface once nothing uses it" is a third. Each step is small enough to check with confidence, and if something goes wrong, it's immediately clear which step introduced it — which is far harder to determine once ten steps' worth of change have landed in a single commit.

## The discipline was always the point

None of this is specific to working with an agent — research, plan, review, implement in stages is exactly how a careful engineer approaches a hard, structural change without an AI involved at all. What's different with an agent is that this discipline has to be stated rather than assumed: an agent asked only to "do the refactor" will happily skip straight to generating a large diff, because nothing in a bare request tells it to slow down. Imposing that structure explicitly — research first, plan for review, implement in reviewable steps — is what separates an agent-driven refactor that goes well from one that quietly accumulates damage no single diff made obvious.
