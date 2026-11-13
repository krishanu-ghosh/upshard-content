---
title: Adversarial training and robust design
description: Baking resistance into a model through adversarial training and constitutional classifiers is the deeper counterpart to a bolted-on guardrail.
roadmap: ai-red-teaming
node: adversarial-training-and-robust-design
group: defense
video:
  provider: youtube
  id: "umfeF0Dx-r4"
  start: 0
  end: null
creator:
  name: FAR.AI
  channel_url: https://www.youtube.com/@FARAIResearch
duration_seconds: 960
level: advanced
prerequisites: [guardrails-and-output-filtering, adversarial-examples]
takeaways:
  - Adversarial training deliberately trains a model on the jailbreak and adversarial examples that currently break it, so resisting that family of attack becomes part of what the model has learned, not a rule checked by a separate system.
  - Anthropic's constitutional classifiers use red-team-generated attack examples to train a defense that generalizes to attacks it wasn't shown directly, rather than memorizing the exact phrasing of known jailbreaks.
  - Because this robustness is baked in at training time, it is far harder to retrofit than a guardrail — you cannot patch it with a new rule, you have to retrain or fine-tune the model itself.
  - Nicholas Carlini's research record shows that nearly every published adversarial defense has eventually been broken by a slightly stronger attacker, so "robust" should be read as resistant to known attacks today, not as a permanent guarantee.
check_questions:
  - q: What does adversarial training actually change about a model, compared to adding a guardrail in front of it?
    a: Adversarial training fine-tunes the model's own weights on examples of the attacks it should resist, so resistance becomes part of the model's learned behavior, whereas a guardrail is a separate system checking inputs and outputs without changing the underlying model at all.
  - q: How do Anthropic's constitutional classifiers try to generalize beyond the exact jailbreaks used to train them?
    a: They are trained on a broad, red-team-generated set of attack examples and the principles behind what is disallowed, so the classifier learns the underlying pattern of an attack rather than memorizing specific phrasings, which helps it catch variations it has never seen.
  - q: Why does Carlini's research conclude that robust design should be treated as an ongoing arms race rather than a solved problem?
    a: Historically, nearly every adversarial defense that looked solid against the attacks known at publication time was later broken by a new, slightly stronger attack, so a defense that resists today's known attacks offers no guarantee against tomorrow's.
deeper:
  - label: "Defending against AI jailbreaks — Anthropic"
    url: https://www.youtube.com/watch?v=BaNXYqcfDyo
related_formulas: []
---

## Training the resistance in, instead of bolting it on

The previous lesson's guardrails solve a real problem cheaply: wrap any model
in a classifier or a rails engine and you catch a large share of attacks
without touching the model itself. But that cheapness is also the ceiling — a
guardrail only recognizes what it was built to recognize, and an attacker who
treats it as a black box to search against, the way Best-of-N does, can often
find the gaps. Adversarial training takes the opposite approach: instead of
inspecting inputs and outputs from the outside, it changes what the model
itself does when it encounters an attack, by including attack examples
directly in training.

Concretely, this means taking a corpus of adversarial examples and jailbreak
attempts — many produced by red-teaming of exactly the kind this roadmap has
spent most of its lessons covering — and fine-tuning the model on them, with
the correct, safe response as the training target. If Crescendo-style
escalation or many-shot jailbreaking reliably broke the model last month,
those exact transcripts, paired with a refusal or a safe completion, become
additional training data. The model that comes out the other side isn't
checking a rule against a list; it has adjusted its own weights so that the
specific class of input that used to produce compliance now produces the
response you want. This is the model-level equivalent of a vaccine: expose
the system to a known form of the threat so it builds its own resistance,
rather than relying on a gatekeeper standing at the door.

## Constitutional classifiers: red-teaming as training data

Anthropic's constitutional classifiers push this idea further by training a
defense model directly on red-team output. The practical workflow is close to
what the automated-red-teaming and tooling lessons elsewhere in this roadmap
describe: generate a large, varied set of attack attempts — some automated,
some human-crafted — targeting a written set of principles about what the
model should and shouldn't produce, then train a classifier on the boundary
between compliant and non-compliant responses that those attacks reveal. The
goal isn't to memorize the literal wording of every jailbreak tried during
red-teaming; a classifier that did only that would fail the instant an
attacker changed one word. The goal is to generalize from a representative
sample of attacks to the underlying pattern those attacks share, so a
jailbreak phrased in a way the classifier never saw during training still
gets caught because it shares the same structural intent as the ones it did
see. That generalization is the entire point of treating red-team output as
training data rather than as a literal blocklist.

## Why "robust" is a moving target, not a finish line

Nicholas Carlini has spent a large part of his career doing one specific,
humbling thing: taking published adversarial-ML defenses and breaking them.
The pattern that emerges across a decade of that work is remarkably
consistent — a defense gets published, it withstands the attacks known at the
time, and within a relatively short window, a slightly adapted or stronger
attack defeats it anyway. Nothing about adversarial training or constitutional
classifiers exempts them from that pattern. They are real, meaningful
improvements over having no model-level robustness at all, and they are
harder to defeat than a guardrail you can poke at from outside, precisely
because the resistance is baked into the weights rather than checkable as a
separate system. But "resists every attack we've tried so far" is a claim
about the past, not a guarantee about the future. The honest engineering
posture, and the one this lesson wants to leave you with, is to treat model
robustness the same way you'd treat any other security property under active
adversarial pressure: valuable, worth investing in, and never finished.
