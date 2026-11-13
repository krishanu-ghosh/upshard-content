---
title: The context window
description: The window as a budget you spend rather than a setting you configure, what happens at the limit, and why a bigger one is not a strategy.
roadmap: ai-engineer
node: the-context-window
group: working-with-models
video:
  provider: youtube
  id: "-uW5-TaVXu4"
  start: 0
  end: null
creator:
  name: Matt Pocock
  channel_url: https://www.youtube.com/@mattpocockuk
duration_seconds: 573
level: beginner
prerequisites: [tokens-and-tokenization]
takeaways:
  - The window is a single budget shared by the system prompt, the conversation history, retrieved documents, tool definitions, tool results and the response.
  - A model is stateless, so every turn re-sends the entire history; a long conversation costs more per turn than a short one even when the new message is identical.
  - Accuracy degrades well before the window is full, so the usable window is smaller than the advertised one.
check_questions:
  - q: Why does the tenth turn of a conversation cost more than the first?
    a: The model keeps no state between calls, so the whole history is re-sent each time. The budget consumed grows with the conversation even if each new message is the same size.
  - q: What is wrong with answering "use a model with a bigger window" to a context problem?
    a: Measured accuracy falls as input grows, unevenly and well before the limit. A bigger window buys room, not reliability, and it costs more per call.
deeper:
  - label: "Chroma — Context Rot: How Increasing Input Tokens Impacts LLM Performance"
    url: https://www.youtube.com/watch?v=TUjQuC4ugak
related_formulas: [kv-cache-size, transformer-forward-flops]
---

## A budget, not a setting

The context window is the total number of tokens a model can attend to in one
call. That total is shared, and the sharing is the part people miss. Your
system prompt, the whole conversation so far, every retrieved document, every
tool definition, every tool result and the response the model is about to
generate all come out of one allowance. This is why context is engineered
rather than configured. Everything you add competes with everything else, and
the thing it most often competes with is the room left for the answer.

## How big windows actually got

Early widely-used chat models offered around four thousand tokens of context —
a few pages of text. That grew to tens of thousands, then to the
hundred-thousand range, and current frontier models advertise context windows
in the hundreds of thousands to over a million tokens, enough in principle to
fit an entire codebase or a short book in a single call. The growth came from
engineering work on the attention mechanism, which naively costs compute and
memory proportional to the square of the sequence length — doubling the input
roughly quadruples the work — so extending the window required real
algorithmic progress, not just bigger hardware.

## What a token in the window actually costs to hold

Behind the scenes, every token processed so far has an associated set of
numbers cached in GPU memory — a key and a value vector, per attention head,
per layer — collectively the **KV cache**, covered with its own sizing
arithmetic on the [KV cache size](/formulas/kv-cache-size) formula page. The
KV cache is what lets the model avoid recomputing earlier tokens on every new
step, and it is also the main reason a long conversation is expensive to
serve: the cache for a hundred-thousand-token conversation can occupy many
gigabytes of GPU memory per concurrent user, which is a hard constraint on how
many long conversations a provider can serve at once, not just a cost
line-item.

## Where it goes empty, concretely

Imagine a modest budget of 32,000 tokens. A system prompt with detailed
instructions and a few examples might run two thousand tokens before any real
work begins. Five retrieved documents at four hundred tokens each is another
two thousand. A conversation ten turns deep, at a few hundred tokens a turn,
adds another few thousand. Tool definitions for a dozen tools can easily cost
another thousand or two. By the time the model is asked to answer, a third of
the nominal budget may already be spent on scaffolding, before the actual
question or the space needed for a thorough answer is counted at all. This
arithmetic is why a system that works fine in a demo with one short exchange
can start truncating, refusing, or quietly losing earlier instructions once
real users carry on longer conversations.

## Why accuracy degrades before the window is full

A model attending to a hundred thousand tokens is not attending to all of them
equally well. Research measuring performance as input length grows — the
"context rot" work referenced in this lesson's deeper link — finds that
accuracy falls steadily and non-uniformly as more tokens are added, and falls
faster still when the relevant information sits in the middle of a long input
rather than near the beginning or the end, a pattern often called
**lost-in-the-middle**. The practical implication is blunt: place the most
important instructions and the most important retrieved facts near the start
or the end of the prompt, and do not assume that because something fits in
the window, the model is reading it with equal attention.

## Why bigger is not the answer

Advertised windows have grown enormously, and the reflex is to treat the
problem as solved — just use the model with the largest window and stop
worrying about what goes in. Measurement says otherwise: performance degrades
as input grows regardless of the nominal limit, and a model that can accept a
million tokens does not reason over a million tokens as well as it reasons
over ten thousand. The practical reading: treat the usable window as a
fraction of the stated one, put the important material where the model will
weight it, and remove what is not earning its place — which is exactly the
discipline covered in
[context engineering](/roadmaps/ai-engineer/context-engineering).
