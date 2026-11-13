---
title: Guardrails and moderation
description: Input and output checks, classifiers and moderation APIs, human review — and an honest account of what guardrails can promise.
roadmap: ai-engineer
node: guardrails-and-moderation
group: safety
video:
  provider: youtube
  id: "O_0QuRqvB5M"
  start: 0
  end: null
creator:
  name: Jumping Rivers
  channel_url: https://www.youtube.com/@jumping-rivers
duration_seconds: 1537
level: intermediate
prerequisites: [prompt-injection, llm-as-judge]
takeaways:
  - A guardrail is a runtime check on the way in or the way out, and it is probabilistic — a filter, not a boundary.
  - Output checks catch what input checks cannot, because the harm is often in what the model produced, not in what was asked.
  - Every guardrail has a false-positive cost, and a system that blocks legitimate requests is failing its users in a way nobody logs.
check_questions:
  - q: Why are output guardrails necessary if you already filter the input?
    a: Because harm often has no harmful input. A benign question can produce a leaked system prompt, an ungrounded claim, or private data pulled in by retrieval — and only an output check sees that.
  - q: What is the cost of tightening a guardrail, and why is it easy to miss?
    a: False positives — legitimate requests blocked. They rarely generate an error or an alert, so the damage shows up as users quietly giving up, not in any metric you're watching.
deeper:
  - label: "AI Engineer — Cheap guardrails with fine-tuned ModernBERT classifiers"
    url: https://www.youtube.com/watch?v=YZHPEkfy2kc
  - label: "Applied with AI — Agent guardrails for injection and PII"
    url: https://www.youtube.com/watch?v=9Ek2cvIHq1c
related_formulas: []
---

## What a guardrail actually is

A check that runs around the model, not inside it. On the way in, it inspects
the request before the model ever sees it. On the way out, it inspects the
generated response before it reaches the user — and can block it, redact
part of it, trigger a regeneration, or escalate to a human.

The word "guardrail" suggests something close to a wall. The reality is a
classifier with a tunable threshold, which means there are always two error
rates to manage at once, and where you set the dial between them is a product
decision as much as a security one.

## The categories worth knowing apart

**Toxicity and abuse filters** catch content that's harmful on its own terms,
regardless of context. **PII detectors** flag personal information — names,
phone numbers, government IDs — that shouldn't go to a third-party API or sit
in storage longer than necessary. **Jailbreak and injection detectors** look
for patterns associated with attempts to override a system's instructions,
covered in [prompt injection](/roadmaps/ai-engineer/prompt-injection).
**Topic restriction** keeps a system from engaging with subjects outside its
intended scope — a legal-document assistant declining to offer medical
advice, say — regardless of whether the content is otherwise harmful.
**Format validation** checks that output actually conforms to what's
expected structurally. It overlaps with the schema enforcement in
[structured output](/roadmaps/ai-engineer/structured-output), but a guardrail
here is usually checking whether the content is semantically appropriate, not
just whether it parses.

## Cheap first, expensive only when needed

A sensible guardrail pipeline runs a fast, cheap classifier — often a small,
purpose-trained model answering in milliseconds — as a first pass over every
request. That catches the large majority of obvious cases at almost no cost.
Only the uncertain or borderline cases get escalated to something slower and
pricier: a bigger model, a more thorough analysis, a full human review.

This is what makes thorough guardrails affordable at real volume. Running the
most careful possible check on every single request would be too slow and too
expensive — the layering means you only pay for that on the minority of cases
that actually need it.

## The trade-off, with real numbers

Every classifier threshold trades false positives against false negatives,
and tightening one side always loosens the other. Say a toxicity filter is
set to catch 95% of genuinely toxic messages — 95% recall. At that
sensitivity it might also flag 3% of entirely benign messages as toxic — 97%
precision. Push recall to 99%, and the false-positive rate might climb to 8
or 10% of benign messages, because catching the hardest, most borderline
toxic cases requires an aggressiveness that also sweeps in more innocent ones.

Which side to favor is a product call, not just a technical one. A system
moderating public content at scale might accept more false positives to keep
toxic content out. An internal tool used by trusted employees might accept
more false negatives rather than interrupt legitimate work.

## Open-source tooling, briefly

A number of open-source projects exist for exactly this layer — small,
purpose-built classifiers for toxicity, PII and jailbreak detection, and
broader frameworks for declaring guardrail policies instead of hand-coding
each check. Worth knowing they exist before you build equivalents from
scratch. But evaluate any of them against your own traffic first, using the
same approach as the [eval set](/roadmaps/ai-engineer/building-an-eval-set)
lesson — a classifier tuned on someone else's data won't automatically
transfer its stated accuracy to yours.

## Human review, done right

For anything irreversible or high-stakes, routing to a human before the
action proceeds is the strongest guardrail available. The review interface
has to show the actual, concrete action — the literal message about to be
sent, the literal transaction about to execute — never a natural-language
summary the model wrote describing its own intent. This matters for the same
reason covered in [prompt injection](/roadmaps/ai-engineer/prompt-injection):
a model whose behavior has been manipulated can write a reassuring summary of
a harmful action just as fluently as an honest one.

## What guardrails can't promise

A guardrail reduces the rate of a bad outcome. It doesn't eliminate it.
Classifiers are themselves models, and can often be evaded by the same
techniques they're built to detect — a pattern familiar from spam filtering
and other adversarial classification. Think of guardrails as defense in
depth, layered on top of an architecture that already bounds the damage a
single failure can cause — never the one thing a system relies on for
safety.

And whatever threshold you pick, measure the false positives actively. A
blocked legitimate request produces a frustrated user and, usually, no log
entry calling attention to it. Sampling what gets blocked and reading it
periodically is the only reliable way to notice a threshold has drifted too
far toward caution.
