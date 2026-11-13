---
title: Chain-of-thought prompting
description: "Asking a model to reason step by step before answering, so its own intermediate tokens become working memory it can condition on."
roadmap: ai-agents
node: chain-of-thought-prompting
group: reasoning-and-planning
video:
  provider: youtube
  id: "zKB6XcNzsS8"
  start: 0
  end: null
creator:
  name: At A Glance!
  channel_url: https://www.youtube.com/@ataglanceofficial
duration_seconds: 1440
level: intermediate
prerequisites: [the-agent-loop]
takeaways:
  - Prompting a model to produce intermediate reasoning steps before its final answer substantially improves accuracy on multi-step arithmetic, logic, and multi-hop questions.
  - "The mechanism is architectural, not stylistic: a model only ever conditions on tokens it has already generated, so writing out intermediate steps gives it a working memory a direct jump to the answer never has."
  - Chain-of-thought helps most on problems with several dependent steps, and barely helps at all on single-step lookups where there is no intermediate state to track.
  - The technique can be triggered with nothing more than an instruction like "think step by step," which makes it one of the cheapest reliability gains available in prompt design.
check_questions:
  - q: Why does writing out intermediate steps actually improve a model's accuracy, rather than just making its answer look more thorough?
    a: A model generates one token at a time and can only condition on tokens already in its context, so when it writes intermediate steps those steps become real inputs to the next token it produces, letting later steps build on earlier conclusions instead of the model having to hold the whole computation in a single hidden pass.
  - q: On what kind of task does chain-of-thought prompting help the least, and why?
    a: It helps the least on single-step lookups, such as "what is the capital of France," because there is no intermediate state to track or build on, so there is nothing for the extra reasoning tokens to condition the final answer on.
deeper:
  - label: "CMU Advanced NLP 2022 (9): Prompting — Graham Neubig"
    url: https://www.youtube.com/watch?v=5ef83Wljm-M
related_formulas: []
---

## The model has no scratchpad unless you give it one

A language model produces its answer one token at a time, and each new token is chosen by looking only at the tokens that already exist — the prompt plus whatever the model itself has written so far. If you ask a model a multi-step question and demand the final answer immediately, it has to arrive at that answer in a single forward pass with no visible intermediate state: no scratch work, no partial sums, nothing to check against. Chain-of-thought prompting exploits a loophole in that constraint. If you instead ask the model to write out its reasoning before the answer, each sentence of reasoning becomes a token the model can condition on when generating the next sentence. The "thinking" is not decoration sitting next to the real computation — for an autoregressive model, the thinking tokens *are* part of the computation, because they become inputs to everything generated afterward.

## A worked example: the same problem, two ways

Consider this word problem: *A train leaves a station at 60 miles per hour. Two hours later, a second train leaves the same station on the same track at 90 miles per hour, chasing the first. How many hours after the second train departs does it catch the first?*

**Direct-answer prompting** — asking only for the final number — frequently produces a wrong answer like "2 hours," arrived at by some garbled shortcut such as dividing the speed difference into the head start distance without properly accounting for both trains' positions as functions of time. The model has to get the whole relationship right in one unbroken pass, and multi-step word problems are exactly where that breaks down.

**Chain-of-thought prompting** — adding "let's work through this step by step" — walks the model through state it can actually use:
1. When the second train departs, the first train has already traveled 60 × 2 = 120 miles.
2. From that moment, the first train's position is 120 + 60t, and the second train's position is 90t, where t is hours since the second train departed.
3. The second train catches the first when 90t = 120 + 60t, so 30t = 120, so t = 4.
4. The second train catches the first 4 hours after it departs.

Each line here is a token sequence the model wrote and then conditions the next line on. Step 2 could not exist without step 1 already being in context; step 3 is just algebra performed on a quantity that step 2 made explicit. The correct answer, 4 hours, falls out of the chain because each piece of intermediate state was written down instead of being held invisibly.

## Why this works at all

Treat the model's forward pass as a fixed amount of computation per generated token. A direct answer compresses an entire multi-step derivation into that one pass for one token, and the model's internal computation simply is not deep enough to reliably chain several dependent inferences together that way, especially on longer or less familiar problems. Spreading the same derivation across many tokens turns one hard, deep problem into a sequence of easier, shallow ones, each conditioning on the output of the last. This is also why chain-of-thought's benefit scales with how many genuinely dependent steps a problem has: a single-step fact lookup has no chain to spread out, so there is nothing for extra reasoning tokens to help with, and asking for step-by-step reasoning on such a question mostly just adds length without adding accuracy.

## Where the benefit shows up and where it does not

The clearest gains appear on arithmetic word problems, multi-hop questions that require combining several retrieved facts, and logic puzzles with several interacting constraints — anything where an answer depends on getting an earlier sub-result right before the next sub-result can be computed. The gains are much smaller on tasks that are really pattern matches or single lookups, and chain-of-thought can even hurt on tasks where verbose reasoning introduces its own errors or where the correct answer depends on an intuition that does not decompose cleanly into written steps. This is the practical reason agent builders reach for chain-of-thought selectively rather than wrapping every prompt in "think step by step": it is a real accuracy tool for compositional problems, not a universal quality switch.

## From a prompting trick to an agent habit

Inside an agent loop, this same idea generalizes beyond isolated questions. Before choosing which tool to call or what to do next, an agent that writes out its reasoning is doing the same thing as the train problem above: making its own intermediate judgment part of the context it conditions its next decision on, rather than jumping straight to an action chosen in one unexamined step. The next lessons in this group build directly on that — interleaving reasoning with real actions, and branching into multiple reasoning paths when one chain is not enough.
