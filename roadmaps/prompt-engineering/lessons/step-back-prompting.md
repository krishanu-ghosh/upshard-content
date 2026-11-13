---
title: Step-back prompting
description: Answer a general question first and a specific one second — abstraction before application reduces reasoning errors.
roadmap: prompt-engineering
node: step-back-prompting
group: advanced-patterns
video:
  provider: youtube
  id: "rdONCqZoUZE"
  start: 0
  end: null
creator:
  name: 1littlecoder
  channel_url: https://www.youtube.com/@1littlecoder
duration_seconds: 666
level: intermediate
prerequisites: [chain-of-thought-prompting]
takeaways:
  - Step-back prompting asks the model for a general principle before asking it to solve the specific problem that needs that principle.
  - Separating "what rule applies" from "how do I apply it" keeps a wrong first move from contaminating the rest of the reasoning.
  - The technique costs one extra round trip, so it pays off on problems that genuinely hinge on picking the right abstraction, not on simple lookups.
check_questions:
  - q: What does step-back prompting ask the model to do before answering the original question?
    a: It asks the model to derive a more general principle or question that the original question is an instance of, and answer that first.
  - q: Why does answering the general question first reduce errors on the specific one?
    a: It isolates the step most prone to mistakes — identifying which principle governs the problem — from the step of applying it, so a correct principle is available as clean context rather than tangled up with arithmetic or detail-tracking.
deeper:
  - label: "Micro Learning — Step-Back Prompting Explained in 5 Minutes"
    url: https://www.youtube.com/watch?v=ucRH4Lws_Pk
related_formulas: []
---

## The idea: zoom out before you zoom in

Step-back prompting comes from a 2023 Google DeepMind paper, "Take a Step Back:
Evoking Reasoning via Abstraction in Large Language Models." The technique is a
two-call pattern. Given a specific question, the first call asks the model a
more general, more abstract version of it — a "step back" question whose
answer is a principle, formula, or piece of background knowledge the specific
question depends on. The second call then answers the original specific
question, with the general answer supplied as extra context.

This sounds like a small reordering, but it targets a real and common failure
mode. When a model answers a hard, specific question directly, it has to
simultaneously figure out *which* rule or fact applies and *how* to apply it
to the details in front of it. If it misidentifies the rule, everything built
on top of that mistake is wrong, even if the arithmetic or logic layered on
top is flawless. Step-back prompting splits those two jobs into two calls, so
a wrong first guess at the rule is cheap to catch and correct before the model
has invested any effort in applying it.

## A worked example

Take a specific physics question: "A particle's pressure is doubled and its
temperature is quadrupled — by what factor does its volume change, assuming
constant amount of gas?" Asked directly, a model has to recognize this is an
ideal-gas-law question, recall the law, rearrange it, and substitute numbers,
all in one pass. If it mis-recalls the relationship or picks the wrong gas
law variant, the final answer is wrong even though every subsequent step was
executed correctly.

With step-back prompting, the first call asks something like: "What physical
principles govern the relationship between pressure, temperature, and volume
for a fixed amount of ideal gas?" That question is easier to answer well
because it has no numbers to juggle and no specific scenario to parse — the
model can recall the ideal gas law, PV = nRT, cleanly. The second call then
receives that law as context alongside the original question, and only has to
substitute the given values and solve. The hard part — remembering the right
relationship — happened in isolation from the error-prone part — applying
it to specific numbers.

## Why abstraction-first works

The underlying claim is that large language models are often capable of
stating a correct general principle but then make a mistake while trying to
apply it under the extra cognitive load of a concrete scenario. By asking for
the abstraction first, you give the model a chance to retrieve the right
knowledge with nothing else competing for its attention, and then hand that
clean answer to the second call as a scaffold. The second call isn't deriving
the principle from scratch anymore — it's doing the comparatively easier job
of plugging values into something it's already been told is correct.

This also makes step-back prompting a natural complement to chain-of-thought
prompting rather than a competitor. Chain-of-thought asks a model to show its
intermediate reasoning steps within a single answer; step-back prompting adds
a separate, prior call whose entire purpose is to establish the right
high-level frame before any chain of reasoning begins. You can combine them:
step back to get the principle, then chain-of-thought through applying it.

## When the extra round trip is worth it

Step-back prompting costs a second model call and the added latency that
comes with it, so it's not worth reaching for on questions where the
applicable rule is obvious or where there's effectively no risk of
misidentifying it. It earns its cost on problems where picking the wrong
frame is a real, common failure — multi-domain word problems, questions that
could plausibly be mistaken for a different but superficially similar
problem type, or tasks where the model has historically shown it can state a
rule correctly in isolation but garbles it when asked to apply it to a messy,
specific case all at once.

A practical way to decide: if you can imagine a wrong answer coming from "the
model used the wrong formula or framework" rather than "the model made an
arithmetic or logical slip while using the right one," step-back prompting is
targeting exactly that risk. If your failures are mostly the second kind,
the extra call won't help as much, and chain-of-thought prompting alone — or
self-consistency on top of it — is the better lever to pull.

## Keeping the two calls honest

One failure mode worth guarding against: if the step-back question is phrased
so specifically that it basically re-derives the answer to the original
question, you've gained nothing but latency. The general question needs to be
genuinely more abstract — a definition, a law, a classification — not a
restatement of the original problem with the numbers removed. A useful test
is whether the answer to the step-back question would be exactly the same for
several different specific problems in the same family; if it would, you've
found the right level of abstraction.
