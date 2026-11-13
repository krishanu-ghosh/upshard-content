---
title: Prompting for Code Changes
description: How to give an AI coding agent the right context, constraints, and scope so a code-change request succeeds on the first or second try.
roadmap: ai-product-builder
node: prompting-for-code-changes
group: building-with-ai-coding-agents
video:
  provider: youtube
  id: "XSZP9GhhuAc"
  start: 0
  end: null
creator:
  name: Anthropic
  channel_url: https://www.youtube.com/@anthropic-ai
duration_seconds: 1767
level: intermediate
prerequisites:
  - working-with-claude-code
  - working-with-cursor-and-copilot
takeaways:
  - A good prompt for a code change gives the agent the context it can't infer on its own — which files are involved, which conventions to follow, which parts of the system are off-limits.
  - "Constraints do more work than instructions: telling an agent what not to touch, what interfaces must stay stable, and what style to match narrows its search space dramatically."
  - Scoping a request to something reviewable — one feature, one bug, one refactor step — keeps the resulting diff small enough to actually check, rather than trusting it because it's too large to read.
  - When the first attempt is wrong, the fastest fix is usually to point at the specific place it went wrong and say why, not to restate the whole request from scratch.
  - "Treat the agent's plan, if it gives one, as a checkpoint: catching a bad approach before implementation is far cheaper than catching it in the diff afterward."
check_questions:
  - q: Why does specifying what an agent should not touch often matter more than specifying what it should do?
    a: An agent has to guess the scope of any underspecified request, and without an explicit boundary it may edit files, interfaces, or behavior you had no intention of changing; naming what's off-limits removes that guesswork directly.
  - q: Why is scoping a request to a small, reviewable change better than asking for a large change in one shot?
    a: A small, scoped diff can actually be read and verified line by line, while a sprawling one-shot change tends to get rubber-stamped simply because reviewing it properly would take too long — which is exactly when mistakes slip through.
  - q: When an agent's first attempt misses the mark, what's usually more effective than re-explaining the whole task?
    a: Pointing at the specific part of the output that's wrong and explaining why, so the agent can correct that one thing using the context it already has, rather than starting over from a restated prompt that may lose the useful parts of the first attempt.
deeper:
  - label: "Agentic Prompt Engineering with Claude Code (For you, your team, and your AGENTS)"
    url: "https://www.youtube.com/watch?v=luqKnexhpFs"
related_formulas: []
---

## Context is the scarce resource

An agent doesn't know your codebase's unwritten rules unless you supply them or it discovers them by reading. "Add input validation to the signup form" is technically a complete sentence, but it leaves open which validation library the project already uses, whether errors should show inline or as a toast, and whether there's an existing validation pattern elsewhere worth copying. Every one of those gaps gets filled by a guess, and guesses are where generated code drifts from house style even when it's functionally correct. The fix isn't necessarily a longer prompt — it's naming the two or three things that actually disambiguate the task: "use the same `zod` schema pattern as `LoginForm.tsx`, and show errors the way `FieldError` already renders them."

## Constraints narrow the search more than instructions expand it

Telling an agent what to do defines a goal; telling it what not to do, or what must stay the same, eliminates whole categories of unwanted solutions before it starts. "Don't change the public API of `UserService`," "keep this under 50 lines," or "the existing tests must keep passing unmodified" each rule out a large space of technically-valid-but-unwanted implementations in a single sentence. This matters more than it sounds like it should, because an agent optimizing for "satisfy the request" will often take the path of least resistance — rewriting an interface, deleting a test that's now inconvenient — unless told explicitly that path is closed.

## Size the request to the size you can review

The single biggest lever over code-change quality isn't prompt wording at all — it's scope. A request sized to "add this one validator," "fix this one bug," or "extract this one function" produces a diff a human can actually read in full and judge with confidence. A request sized to "refactor the auth module" produces a diff large enough that reviewing it properly takes real effort, which creates pressure to skim it instead — and skimming is exactly how a subtle, wrong, or unsafe change gets merged. If a task feels too big to prompt in one shot, that's a signal to break it into steps, not a reason to write a more elaborate single prompt.

## Iterating on a miss

When an agent's first attempt is off, the efficient move is almost never to restart with a longer, more careful version of the original prompt. It's to point at exactly what's wrong — "this throws when the input is empty, handle that case" — and let the agent fix that one thing in the context it's already built up. Restating the whole task from scratch discards the useful parts of the first attempt along with the flawed part, and often reproduces the same mistake because the ambiguity that caused it is still there. Precise, localized correction converges faster than a fresh, more verbose attempt at the same underspecified request.

## Reviewing the plan before the diff

For anything beyond a small edit, many agents will describe their intended approach before touching files — which functions they'll change, in what order, with what tradeoffs. That plan is a cheap place to catch a wrong direction. Reading five lines of planned approach and saying "don't touch the database layer for this, there's a simpler way" costs a few seconds; catching the same mistake after it's already been implemented across six files costs a full review cycle and a rewrite. Treating the plan stage as a real checkpoint, not a formality to skip past, is one of the most valuable habits in working with an agent on anything non-trivial.
