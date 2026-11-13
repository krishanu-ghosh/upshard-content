---
title: Safety filter and guardrail bypasses
description: Guardrails are a separate system bolted around a model — bypassing them, via tricks like Skeleton Key or Best-of-N, is a distinct problem from jailbreaking.
roadmap: ai-red-teaming
node: safety-filter-and-guardrail-bypasses
group: prompt-attacks
video:
  provider: youtube
  id: "JDjYLXJqsiA"
  start: 0
  end: null
creator:
  name: Donato Capitella
  channel_url: https://www.youtube.com/@donatocapitella
duration_seconds: 1020
level: intermediate
prerequisites: [jailbreak-techniques]
takeaways:
  - A guardrail is a separate system wrapped around the base model — an input classifier, an output moderation pass, or a dedicated safety model — so defeating it is a different problem from jailbreaking the base model's own trained alignment, even though attackers routinely chain both.
  - Skeleton Key defeats a guardrail by getting the model to satisfy its literal rule, such as prefacing unsafe content with a warning label, while completely defeating the rule's actual intent.
  - Best-of-N treats the guardrail as a black box and searches against it, resampling or lightly perturbing a prompt many times and keeping whichever variant happens to slip through.
  - Automated, repeated probing like Best-of-N scales a single bypass idea into thousands of attempts, which is exactly the capability the methodology group's automated red-teaming tools are built around.
check_questions:
  - q: Why is bypassing a guardrail a conceptually different problem from jailbreaking a model's base alignment?
    a: A guardrail is a separate system layered around the model, such as an input classifier or a dedicated moderation model, so defeating it means fooling that separate system's judgment, whereas jailbreaking targets the base model's own trained refusal behavior directly — an attacker can defeat one without touching the other, though chaining both is common.
  - q: How does the Skeleton Key technique work?
    a: It gets the model to comply with a disallowed request by having it attach a disclaimer or warning label to the output, which technically satisfies a guardrail rule framed as "label unsafe content" while completely defeating that rule's actual purpose of preventing the unsafe content from being produced at all.
  - q: What makes Best-of-N an effective and scalable guardrail bypass?
    a: It treats the guardrail as a black box, resampling or slightly perturbing the same underlying request many times over and automatically keeping whichever variant happens to slip past the filter, which turns a single bypass idea into an automated search that can run thousands of attempts without a human crafting each one.
deeper:
  - label: "Episode 6: Prompt Injection Attacks: Multi-Turn | AI Red Teaming 101 — Microsoft Developer"
    url: https://www.youtube.com/watch?v=HQadhDsQKgY
related_formulas: []
---

## A layer, not a property

The previous lesson treated jailbreaking as an attack on the base model's own trained refusal behavior — the alignment baked in during training. Many production systems add a second, independent layer on top of that: a guardrail. A guardrail isn't part of the model's weights. It's a separate piece of software wrapped around the model — an input classifier that screens a prompt before it reaches the model, an output moderation pass that screens a response before it reaches the user, or a dedicated safety model like Llama Guard whose only job is deciding whether a piece of text is unsafe. This distinction matters because it changes what "bypass" even means. Jailbreaking defeats the model's own judgment about a request. Bypassing a guardrail defeats a different system's judgment about the model's input or output, one the model itself might have refused to engage with even if the guardrail weren't there, or might happily comply with once the guardrail lets it through. Attackers routinely chain the two: use a jailbreak to get the base model to produce something unsafe, then separately sneak that output past the moderation layer sitting in front of the user.

## Skeleton Key: satisfying the letter, not the intent

Skeleton Key, documented by Microsoft, is a guardrail bypass built around a
specific weak spot in how many safety systems are configured: a rule that says
unsafe content must be clearly labeled with a warning rather than a rule that
says unsafe content must never be produced at all. An attacker frames their
request so that the model, instead of refusing, generates the disallowed
content but prefaces it with something like "the following is provided for
context and includes potentially sensitive material." Technically, the letter
of a rule like "flag unsafe output with a warning" has been satisfied. The
actual intent of that rule — keeping the content from reaching the user in
usable form — has been completely defeated, because a warning label does
nothing to stop the information itself from being delivered. This is the
general shape of a huge class of guardrail bypasses: find the gap between what
a rule literally checks for and what it was actually meant to prevent, and
produce output that satisfies the first while violating the second.

## Best-of-N: treating the filter as a black box to search

`Best-of-N` takes a completely different approach that doesn't require understanding why a guardrail fails at all. Instead of crafting one clever prompt, an attacker generates many variants of the same underlying request — different phrasings, capitalization, spacing, or other surface-level perturbations — fires all of them at the target, and keeps whichever variant the guardrail happened to let through. No single variant needs to be understood in advance as "the one that works." The guardrail is treated purely as a black box with a pass/fail output, and the attack is a search over its decision boundary, not an argument about why that boundary is flawed. This works precisely because most guardrails, especially fast ones built for production latency budgets, are imperfect classifiers with real false-negative rates. Any classifier with a nonzero false-negative rate can eventually be found by sampling enough variants — the same statistical logic behind brute-force attacks on any imperfect filter.

## Why automation changes the threat model

Best-of-N's real danger isn't any individual bypass — it's that the technique
is trivially automatable. An attacker doesn't need to be a skilled prompt
engineer; they need a script that can resample a request hundreds or thousands
of times and check which attempts got through, which turns guardrail bypass
from a bespoke craft into a search problem that compute can simply grind
through. That same insight — treating a model-adjacent system as a black box
and probing it automatically at scale — is exactly the foundation the
methodology group later in this roadmap builds its automated red-teaming
lesson on. A red team evaluating a guardrail has to assume an adversary with a
script and time, not just an adversary with one clever idea, because a filter
that blocks every manually crafted bypass a tester thinks of can still fail
against ten thousand automated variants of the same underlying request.

## What this means for defenders

Treating guardrails as a meaningfully separate, auditable layer — rather than
folding "safety" into one vague property of "the AI" — is what makes both
Skeleton Key and Best-of-N useful findings rather than vague anxieties. A team
that knows their output filter checks for the presence of a warning label,
rather than the actual safety of the underlying content, can fix that specific
rule. A team that knows their input classifier has a measurable false-negative
rate under repeated sampling can budget for rate limiting, logging, and
anomaly detection on top of the classifier itself, rather than trusting the
filter alone to hold.
