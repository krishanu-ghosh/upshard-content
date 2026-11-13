---
title: Model extraction and weight stealing
description: Model extraction clones a deployed model's accuracy or exact behavior purely from its API responses, without ever touching its weights.
roadmap: ai-red-teaming
node: model-extraction-and-weight-stealing
group: model-attacks
video:
  provider: youtube
  id: "PPU2a_A2ScI"
  start: 0
  end: null
creator:
  name: USENIX
  channel_url: https://www.youtube.com/@UsenixOrg
duration_seconds: 660
level: advanced
prerequisites: [the-owasp-top-10-for-llm-applications]
takeaways:
  - Model extraction clones a deployed model's behavior purely from its input-output pairs, so the attack surface exists wherever an API answers queries, even with the weights and training data fully hidden.
  - Accuracy extraction aims for a substitute that performs well on the task, while fidelity extraction aims for a substitute that reproduces the original's exact decision boundary, including its specific mistakes.
  - A faithful stolen copy lets an attacker develop adversarial inputs or jailbreak attempts offline, without rate limits or logging, before firing only the finished attack at the real target.
check_questions:
  - q: What is the difference between accuracy extraction and fidelity extraction?
    a: Accuracy extraction produces a substitute that performs about as well as the original on the task, while fidelity extraction produces a substitute that reproduces the original's specific decision boundary and mistakes, which requires more targeted querying near ambiguous inputs.
  - q: Why would an attacker bother stealing a model instead of just using the stolen copy directly?
    a: A local copy removes the rate limits, cost, and logging of the real API, letting the attacker develop and refine other attacks, like adversarial examples or jailbreaks, offline before aiming the finished version at the real deployed model.
deeper: []
related_formulas: []
---

## Turning queries into training data

A deployed machine learning model exposed through an API looks closed: no one
outside the team that trained it can see the weights, the architecture, or
the training set. Model extraction — also called model stealing — shows that
"closed" is the wrong word. If a system answers arbitrary inputs with its
predictions, whether that's a label, a probability vector, or just a ranked
top answer, an attacker can treat the API as a labeling oracle. They send a
large number of inputs, record what the model returns, and use those
input-output pairs as a training set for a second model built entirely by the
attacker. The attacker never touches the original model's parameters,
training data, or source code. The substitute is grown purely from watching
the original answer the exact questions it was deployed to answer — which is
also the thing the business has to keep doing to serve real users. Research
by Matthew Jagielski, Nicholas Carlini, and collaborators, presented at
USENIX Security 2020, formalized this as a continuum of related attacks
rather than a single technique, and that framing matters more than any one
trick in the paper.

## Accuracy extraction versus fidelity extraction

The paper's central distinction is what an attacker is actually trying to
copy, because "a copy of the model" can mean two different goals with very
different query strategies behind them. Accuracy extraction aims for a
substitute that performs about as well as the original on the underlying
task — a competitor doesn't care whether their model's decision boundary
matches the original's; they care whether it classifies images or scores
applications about as accurately. This can sometimes be reached with
generic, task-representative queries, since the goal is matching ground
truth rather than matching the target model specifically. Fidelity
extraction is a harder, narrower goal: a substitute that reproduces the
original's actual behavior, including its idiosyncratic mistakes and its
particular failure modes on edge cases. A fidelity-extracted copy doesn't
just get similar answers right — it gets the same answers wrong, the same
way, on the same inputs. That distinction drives query strategy: probing
near the decision boundary is far more informative for fidelity extraction,
since generic in-distribution samples mostly confirm what the model already
gets easily right.

## Why the copy is the dangerous part, not just the theft

The commercial motive is the obvious one: if a product is a model served
only through a metered API, a sufficiently faithful substitute lets a
competitor undercut the pricing model without replicating the data
collection, labeling, and training investment behind it. That alone is a
real incentive — model extraction is a documented class of intellectual
property loss, not a hypothetical.

The more consequential use, though, is as a stepping stone. A model served
through an API usually sits behind rate limits, per-call cost, and logging
that can flag abnormal query patterns, all of which make interactive attack
development slow and risky. Once an attacker has a local substitute that
approximates the real model closely enough, every subsequent attack —
crafting adversarial inputs, probing for jailbreaks, searching for a trigger
phrase a poisoned version might respond to — can be developed offline, with
unlimited queries, no cost, and no logging, against the substitute. Only the
final, refined attack needs to touch the real target, and it often transfers
because the substitute approximates the same decision boundary. This
transferability is exactly why fidelity extraction is the more dangerous
goal for a red teamer to think about: it isn't about getting a usable model,
it's about getting a usable proxy for attacking the real one.

## What this changes about defending an API

Because the attack lives entirely in the query interface, defenses are
necessarily about query economics rather than about the model itself: rate
limiting, detecting query distributions that look like systematic
boundary-probing rather than real usage, returning less information per
response — a single label instead of a full probability vector, for
instance — and watermarking outputs to later demonstrate that a competitor's
model derived from queries against yours. None of these make extraction
impossible; they raise its cost, which is the recurring theme across most
defenses this roadmap covers later.
