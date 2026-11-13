---
title: Automatic prompt engineering
description: Letting an LLM generate and score candidate prompts against your eval set, and DSPy's compiler-driven version of the same idea.
roadmap: prompt-engineering
node: automatic-prompt-engineering
group: optimization
video:
  provider: youtube
  id: "5_BFqmQv1po"
  start: 0
  end: null
creator:
  name: "At A Glance!"
  channel_url: https://www.youtube.com/@ataglanceofficial
duration_seconds: 1545
level: intermediate
prerequisites: [evaluation-and-testing]
takeaways:
  - Automatic Prompt Engineer (APE) uses an LLM to propose candidate instructions, scores each against a validation set, and keeps the best.
  - DSPy generalizes this into a compiler that optimizes prompts and few-shot examples as tunable parameters against a metric you define.
  - Automation replaces manual iteration, not the eval set — without a real metric and labeled examples, there is nothing for either approach to optimize against.
check_questions:
  - q: What does Automatic Prompt Engineer actually automate, and what does it still require from you?
    a: It automates generating and comparing candidate instruction text using an LLM. It still requires you to supply a validation set and a scoring function that defines what a "better" prompt means for your task.
  - q: In DSPy's model, what is treated as a tunable parameter instead of hand-written text?
    a: The instructions and the few-shot examples inside a prompt are both treated as parameters that an optimizer searches over, in the same way a training loop searches over model weights, driven by a metric rather than hand-editing.
deeper:
  - label: "Connor Shorten — DSPy Explained!"
    url: https://www.youtube.com/watch?v=41EfOY0Ldkc
related_formulas: []
---

## Hand-iterating on a prompt doesn't scale

Every technique earlier in this roadmap — few-shot examples, chain-of-thought, system
prompting — still leaves a human in the loop, rewriting and re-testing phrasing by
hand. That works for a handful of prompts. It breaks down once an application has
dozens of prompts across different tasks, or once a prompt needs to keep working as the
underlying model changes. Automatic prompt engineering replaces the human-in-the-loop
rewrite step with a search procedure: generate candidates, score them against real
examples, keep what wins.

This only works if the "score them against real examples" half already exists — which
is exactly what the [evaluation and testing](/roadmaps/prompt-engineering/evaluation-and-testing)
lesson covers. Automating prompt writing without an eval set just automates guessing
faster; it doesn't remove the need to define what "better" means.

## The original idea: Automatic Prompt Engineer (APE)

APE, from Zhou et al.'s 2022 paper, treats instruction-writing itself as a search
problem over natural language. Given a handful of example input/output pairs for a
task, an LLM is asked to propose candidate instructions that would produce those
outputs — effectively, reverse-engineering plausible phrasings of the task. Each
candidate instruction is then run against a held-out validation set, scored by how well
it actually produces the correct outputs, and the highest-scoring candidate is kept.
Later work extends this into an iterative loop: take the current best prompt, generate
variations around it, re-score, and repeat — the same propose-then-select cycle,
applied more than once.

The mechanism doing the work is ordinary LLM generation plus evaluation; nothing here
trains new parameters or touches model weights, unlike [prompt tuning](/roadmaps/prompt-engineering/prompt-tuning).
What changes is who writes the candidate text — an LLM proposing instructions, scored
automatically — instead of a person guessing and manually re-testing each version.

## The modern, production version: DSPy

DSPy takes the same underlying idea and turns it into a framework rather than a
one-off technique. Instead of writing a prompt directly, you declare a **signature** —
the task's inputs and outputs, described in a few words, like `question -> answer` or
`document -> summary` — and let DSPy's compiler figure out the actual prompt text and
few-shot examples that get the underlying model to satisfy that signature well. The
prompt and the examples attached to it are treated as parameters an optimizer searches
over, the same conceptual move APE makes, but generalized across an entire pipeline of
calls rather than one instruction at a time, and packaged with ready-made optimizer
algorithms rather than a bespoke script.

Running DSPy's compiler still means supplying a metric function (how do you score one
output as better than another?) and a set of training examples the optimizer can try
candidates against. Swap the target model — move from one provider's model to
another's — and recompiling against the same metric and examples re-searches for a
prompt suited to the new model, rather than hoping a hand-tuned prompt for one model
happens to transfer to another.

## What automation does and doesn't buy you

Both approaches remove the slowest, most repetitive part of prompt engineering: writing
a variant, running it, reading the output, deciding if it's better, and doing that
again. A search procedure can try far more variations, far faster, and more
consistently than a person iterating by hand, and it keeps a record of exactly why one
candidate beat another — the score, not a gut feeling.

What neither approach removes is the need for the two things that actually define the
task: a metric that captures what "good" means for your use case, and a set of real
examples to test candidates against. Automatic prompt engineering is a faster search
through prompt-space, not a replacement for doing the evaluation work first. If a team
hasn't built an eval set yet, that is the actual prerequisite — automating the search on
top of a vague or missing definition of success just produces a prompt that is
confidently optimized for the wrong thing.
