---
title: Choosing a model
description: Capability, context window, latency, and cost are the four questions that decide which model a prompt should target.
roadmap: prompt-engineering
node: choosing-a-model
group: model-fundamentals
video:
  provider: youtube
  id: "pYax2rupKEY"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 416
level: beginner
prerequisites: [how-llms-work]
takeaways:
  - A model choice is a trade-off between task capability, context window, latency, and cost — not a search for a single "best" model.
  - Whether a model is instruction-tuned changes how a prompt should be written, since a raw base model continues text rather than following commands.
  - Naming a specific model in your prompting strategy is a decision you will revisit; the criteria for choosing one age far more slowly than any model does.
check_questions:
  - q: Why does it matter whether a model is "instruction-tuned" before you write a prompt for it?
    a: A base model is trained only to continue text, so a command like "Summarize this" may just get continued rather than obeyed. An instruction-tuned (chat) model has been further trained to treat input as a request it should fulfil, which is what makes direct prompting work at all.
  - q: You need to classify 50,000 support tickets into five categories as cheaply as possible. Which of the four axes (capability, context, latency, cost) dominates the decision, and why?
    a: Cost and latency dominate. The task is narrow and well-defined, so a small, cheap, fast model that is merely competent at classification will do the job at a fraction of the price and time of a frontier model, whose extra capability is wasted on a task this simple.
deeper:
  - label: "Valohai — Choosing the Best LLM for Your Use Case (Structured Evaluation Demo)"
    url: https://www.youtube.com/watch?v=DA0CMaw755M
related_formulas: []
---

## The question underneath "which model"

Every prompt you write is written for a specific model, and the same prompt can
behave differently on two different ones — one follows your formatting
instruction precisely, another ignores it half the time; one handles a
32,000-token document, another truncates it. Picking a model is not a
one-time decision made before prompting starts; it is part of prompt
engineering, because the model you target shapes how you should write the
prompt itself. Four questions settle it: can the model do the task at all,
does your input fit in its context window, how fast does it need to respond,
and what can you afford to pay.

## Capability: matched to the task, not maximized

The temptation is to always reach for the most capable model available. Resist
it. A frontier model's extra reasoning ability is wasted on tasks that do not
need it — extracting a date from an email, classifying a support ticket,
rewriting a sentence to be more formal. These are tasks a small, cheap model
handles correctly essentially all the time. Capability matters when a task
requires multi-step reasoning, handling ambiguous or adversarial input, or
following a long, intricate instruction set without dropping a constraint
partway through. The practical approach is to start with the cheapest model
that could plausibly work, test it against real examples of your task, and
only move up in capability when it actually fails — not when it merely seems
underpowered on paper.

## Context window: does the input actually fit

A model's context window is the maximum number of tokens it can hold in a
single call — your system prompt, the conversation history, any retrieved
documents, and the model's own output, all counted together (see
[tokens and context](/roadmaps/prompt-engineering/tokens-and-context)). If
your use case means stuffing in a long document, a large codebase, or a long
running conversation history, the context window is a hard constraint before
any other quality consideration: a model that cannot hold your input cannot
do your task, no matter how capable it is otherwise. Context windows have
grown enormously across model generations, but a larger window is not free —
very long inputs tend to degrade a model's attention to the middle of the
context, a failure mode sometimes called "context rot," so a bigger window
solves the fitting problem without automatically solving the quality problem.

## Instruction-tuned versus base

A base model is trained on a simple objective: predict the next token of
whatever text came before it. Given "Translate this to French:", a base model
may just continue with more English text, because nothing in its training
taught it that an instruction is something to obey rather than continue. An
instruction-tuned (or "chat") model has gone through an additional training
stage — typically supervised fine-tuning on example instructions and
responses, often followed by reinforcement learning from human or AI feedback
— that teaches it to treat an instruction as a request and a conversation as
alternating turns. Nearly every model you will prompt directly today is
instruction-tuned, but the distinction still matters: it explains why the
same wording that works perfectly on a chat-tuned model can fail silently on
a raw base model, and it is the reason techniques like
[few-shot prompting](/roadmaps/prompt-engineering/few-shot-prompting) — showing
examples instead of just instructing — exist, since examples work on base
models too.

## Latency and cost, together

Latency (how long a response takes) and cost (what it costs per token) tend
to move together — a model that produces a token faster than another is
usually also cheaper to run that token through, since both scale with the
model's size and the amount of compute a provider spends per request. The
two pull in different directions depending on what you are building: a
real-time chat interface needs low latency more than it needs the deepest
possible reasoning, while an overnight batch job that re-processes a million
records can tolerate a slower, cheaper-per-call model as long as the per-token
cost is low. A common pattern that resolves the tension is routing — sending
every request first to a small, fast, cheap model, and escalating only the
requests it cannot confidently handle to a larger one, which means a majority
of traffic gets the speed and cost of a small model while only the hard tail
pays the price of a large one.

## Treat the choice as provisional

Whatever model you pick today will not be the one you are prompting against
in a year. New models ship constantly, pricing shifts, and a model that was
the best fit for your task six months ago may now be beaten on every axis by
something newer and cheaper. The criteria in this lesson — capability,
context, latency, cost, and whether a model is instruction-tuned — change far
more slowly than any specific model's name, which is exactly why they are
worth internalizing rather than memorizing today's leaderboard. Writing
prompts that rely as little as possible on one model's specific quirks, and
re-testing a prompt whenever you do switch models, is cheaper in the long run
than optimizing hard for whatever happens to be current right now.
