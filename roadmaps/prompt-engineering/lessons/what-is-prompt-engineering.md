---
title: What is prompt engineering?
description: Why the words you choose change what a language model produces, and what separates a disciplined prompt from a lucky guess.
roadmap: prompt-engineering
node: what-is-prompt-engineering
group: foundations
video:
  provider: youtube
  id: "jC4v5AS4RIM"
  start: 0
  end: null
creator:
  name: Jeff Su
  channel_url: https://www.youtube.com/@JeffSu
duration_seconds: 510
level: beginner
prerequisites: []
takeaways:
  - A prompt is the only input you control into a system whose output is a probability distribution, not a fixed function call.
  - Vague instructions get vague, model-chosen answers; specific instructions about task, context, and format narrow that distribution toward what you actually want.
  - Prompt engineering is an empirical discipline — you change the input, observe the output, and iterate — not a collection of magic phrases.
check_questions:
  - q: Why do two people asking an LLM "the same question" in different words often get meaningfully different answers?
    a: The model has no access to intent, only to the literal text. Different phrasing points it toward different regions of its training distribution, so wording is not cosmetic — it is the entire interface.
  - q: What distinguishes prompt engineering from just typing a question into a chat box?
    a: Deliberately specifying task, context, constraints, and output format, then testing and revising based on what the model actually returns, rather than accepting the first response as final.
deeper:
  - label: "freeCodeCamp.org — Prompt Engineering Tutorial: Master ChatGPT and LLM Responses"
    url: https://www.youtube.com/watch?v=_ZvnD73m40o
related_formulas: []
---

## An instruction to a guesser, not a function call

A large language model does not execute your prompt the way a program
executes code. Underneath, it is a next-token predictor: given everything it
has seen so far, it computes a probability distribution over what token comes
next, samples one, and repeats. Your prompt is not a command with a
guaranteed effect — it is the entire context the model has to decide what
"coming next" should look like. Change the words and you change the
distribution, even when the underlying intent in your head hasn't moved at
all. This is the single fact that makes prompt engineering a real skill
rather than a formality: the interface between what you want and what you
get is made entirely of language, and language is ambiguous by nature.

Ask a model to "summarize this report" and you'll get *a* summary — but
length, tone, and what counts as important are all underspecified, so the
model fills the gaps with whatever is statistically typical for "summary" in
its training data. Ask instead for "a three-bullet summary for a
non-technical executive, each bullet under twenty words, focused on
financial impact" and you've removed most of that ambiguity. The model
didn't get smarter between those two prompts. You gave it a sharper target.

## Why the same intent produces different results

This is the part that surprises people coming from traditional software: a
prompt that worked yesterday can behave differently today if the model
version changed underneath it, and a prompt that works for one task can fail
on a superficially similar one. That's not a bug to be engineered away — it's
a consequence of the underlying mechanism. The model has no persistent
understanding of "what you meant last time." Every call starts fresh, and
every call is a fresh sample from a distribution shaped by exactly the text
you provided, nothing more.

Three things reliably narrow that distribution toward what you want:

**Explicit task framing.** "Write a product description" is weaker than
"write a 50-word product description for an e-commerce listing, emphasizing
durability." The model can only optimize for what you've named.

**Context the model doesn't otherwise have.** It knows general English and
general facts up to its training cutoff. It does not know your company's
style guide, your customer's prior complaint, or which of three similar
documents you mean unless you supply that in the prompt itself.

**A stated output format.** If you need JSON, a table, or a specific
structure, say so explicitly and ideally show an example. Leaving format
unstated means the model guesses, and its guess is whatever shape was most
common for similar requests in its training data — which is frequently not
the shape your downstream code expects.

## What makes prompt engineering a discipline, not a trick

There's a persistent myth that prompt engineering is a bag of secret
phrases — "think step by step," "you are an expert," magic words that unlock
better performance. Some of those phrases genuinely do help on certain tasks
(you'll see exactly why when this roadmap covers chain-of-thought and
role prompting later), but treating them as incantations misses the point.
The actual discipline is empirical: form a hypothesis about what the model
needs to succeed at your task, write a prompt that supplies it, run the
prompt, inspect the output against your success criteria, and revise. That
loop — write, run, evaluate, revise — is prompt engineering. It looks more
like debugging or experimental science than like memorizing syntax.

This matters because it means prompt engineering skill transfers across
models and tasks in a way that memorized phrases don't. A prompt engineer
who understands *why* giving the model an example output helps (it
constrains the distribution toward that shape) can apply the same reasoning
to a brand-new model release or an unfamiliar task, where someone who only
knows "add the word 'expert'" has nothing to fall back on.

## What "good" looks like, before the vocabulary

At this stage you don't yet have the formal vocabulary this roadmap will
build — zero-shot versus few-shot, chain-of-thought, system prompting,
structured output constraints. But you can already evaluate a prompt by
asking three questions: Does it state the task unambiguously? Does it supply
every piece of context the model needs but doesn't already have? Does it
specify the output shape you actually need downstream? A prompt that
answers yes to all three will usually outperform a longer, more
elaborately-worded one that answers no to any of them. Everything that
follows in this roadmap is really a set of techniques for answering those
three questions more precisely and for harder tasks.
