---
title: Chain-of-thought prompting
description: Asking a model to reason step by step before answering, and why that measurably improves accuracy on multi-step problems.
roadmap: prompt-engineering
node: chain-of-thought-prompting
group: basic-techniques
video:
  provider: youtube
  id: "-pSKPW6nRSw"
  start: 0
  end: null
creator:
  name: NStatum
  channel_url: https://www.youtube.com/@nstatum8290
duration_seconds: 287
level: beginner
prerequisites: [few-shot-prompting]
takeaways:
  - Chain-of-thought prompting asks the model to produce intermediate reasoning before its final answer, rather than jumping straight to a conclusion.
  - It helps most on tasks with several dependent steps — arithmetic, multi-step logic, planning — and helps little or not at all on tasks that are really a single lookup or classification.
  - The reasoning text gives the model computational "room" to work through a problem across multiple forward passes, rather than forcing the whole answer out in one.
check_questions:
  - q: Why does asking a model to "think step by step" change its accuracy, not just its output length?
    a: It changes what the model has to produce before committing to a final answer. Generating intermediate reasoning spreads the problem across several token-by-token steps instead of forcing a correct answer out in a single pass with no working space.
  - q: On what kind of task does chain-of-thought prompting provide little benefit?
    a: Tasks that are really a single lookup, classification, or simple transformation, where there is no multi-step reasoning to work through and so no intermediate steps for the model to benefit from writing out.
deeper: []
related_formulas: []
---

## Letting the model show its work

Chain-of-thought prompting asks a model to produce its reasoning before
giving a final answer, instead of jumping straight to a conclusion. The
simplest version is a single added phrase: "think step by step" or "explain
your reasoning before you answer." A more reliable version demonstrates the
pattern directly inside a few-shot example, showing the reasoning text itself
as part of the worked example rather than just asking for it in the
instruction.

Compare these two prompts on the same arithmetic word problem:

> A store had 23 shirts. It sold 8 and received a new shipment of 15. How
> many shirts does it have now? Answer:

A direct-answer prompt like this asks a model to produce the final number in
one step, with no intermediate structure to check its own work against.
Models get problems like this wrong more often than the arithmetic alone
would suggest, especially as the number of steps grows.

> A store had 23 shirts. It sold 8 and received a new shipment of 15. How
> many shirts does it have now? Think step by step, then give the final
> answer.

Asked this way, the model typically produces something like "23 − 8 = 15
shirts remain, then 15 + 15 = 30 shirts" before stating 30. The arithmetic is
no easier in principle, but writing out each step means the model only has
to get one small operation right at a time, and each step is conditioned on
the correctly computed step before it rather than on the original problem
alone.

## Why writing it out actually helps

A model produces its response one token at a time, and every token it has
already written becomes part of what conditions the next one. Forced to
answer in one step, a multi-stage problem has to be solved entirely within
the "thinking" that happens before the first output token — work that never
gets written down and so can't be checked or corrected. Chain-of-thought
prompting turns that invisible, single-shot computation into a sequence of
visible intermediate results, each one feeding into the next token
prediction as context. The model isn't "smarter" when prompted this way; it's
being given room to spread a hard computation across many easier ones instead
of being asked to collapse it into a single guess.

## Where it does not help

Chain-of-thought is not a general-purpose accuracy booster. On a task that is
genuinely a single step — classifying a sentence's sentiment, extracting a
date from text, answering a direct factual lookup — there is no multi-step
reasoning to externalize, so adding "think step by step" mostly adds tokens
and latency without changing the answer. Worse, on some simple tasks it can
actively hurt: a model asked to reason about something trivial can talk
itself into an overcomplicated or hedged answer where a direct response would
have been both faster and correct. The technique earns its keep specifically
on tasks with real intermediate structure: arithmetic with multiple
operations, logic puzzles with several constraints, multi-hop questions where
the answer depends on combining several facts, or planning tasks with
ordered steps.

## Combining it with examples

Chain-of-thought reasoning can be requested with a plain instruction, but it
is noticeably more reliable when demonstrated inside a few-shot example — showing
the model one or two worked problems where the example's "answer" is itself
the full reasoning chain followed by the final result. This fuses the two
previous lessons: few-shot prompting supplies the pattern to imitate, and
that pattern now includes the reasoning steps, not just the final answer
format. A model shown exactly how much detail to reason through, and in what
style, tends to produce more consistent and more correct chains than one
simply told to "think step by step" with nothing demonstrating what a good
chain of thought actually looks like.

## What comes next

Chain-of-thought produces one line of reasoning per attempt, and that line
can still go wrong — a single arithmetic slip early in the chain propagates
into a wrong final answer with no way to catch it. The next two lessons in
this roadmap, self-consistency and tree-of-thoughts, both build directly on
this idea by generating and comparing multiple reasoning paths rather than
trusting a single one.
