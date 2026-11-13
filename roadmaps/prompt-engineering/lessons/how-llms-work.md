---
title: How LLMs work
description: Next-token prediction, the mechanism behind every capability and every failure mode a large language model has.
roadmap: prompt-engineering
node: how-llms-work
group: foundations
video:
  provider: youtube
  id: "LPZh9BOjkQs"
  start: 0
  end: null
creator:
  name: 3Blue1Brown
  channel_url: https://www.youtube.com/@3blue1brown
duration_seconds: 478
level: beginner
prerequisites: []
takeaways:
  - An LLM computes a probability distribution over its next token given everything before it, then samples from that distribution — it does not look up or reason toward an answer the way a database or calculator would.
  - Every ability the model has (coherent writing, following instructions, even apparent reasoning) and every failure it has (hallucination, inconsistency) follows from this one mechanism applied token by token.
  - What looks like "understanding" in an LLM is really an extremely good statistical model of what text plausibly follows other text, learned from an enormous amount of human-written text.
check_questions:
  - q: What does an LLM actually compute at each step of generating a response?
    a: A probability distribution over every possible next token, conditioned on everything generated or provided so far, from which the next token is sampled.
  - q: Why can the same prompt produce a different answer each time you run it?
    a: Because generation involves sampling from a probability distribution rather than looking up a fixed answer, and that sampling has a random component unless it's set to always pick the single most likely token.
deeper: []
related_formulas: []
---

## One operation, repeated

Everything a large language model does — writing an essay, answering a
question, generating code — reduces to one repeated operation: given a
sequence of tokens (roughly, word pieces), predict the probability
distribution over what token comes next, pick one, append it, and repeat.
That's it. There is no separate "reasoning module" and no internal database
of facts it queries. The model is a single enormous function, trained on
a vast amount of text, that has learned to estimate "given this much text,
what plausibly comes next?" extremely well. Generating a whole response is
just running that function over and over, each time feeding the growing
output back in as additional input.

This sounds almost too simple to produce the behavior you actually see from
modern models, and that's the right reaction to have — it's the thing worth
sitting with. The sophistication isn't in a second mechanism layered on top
of next-token prediction; it's in how good the prediction is after training
on a meaningful fraction of all the text humans have put online. Predicting
the next word well enough, across enough examples of explanation,
argument, code, and conversation, turns out to require the model to encode
an enormous amount of structure about grammar, facts, and even patterns that
look like step-by-step reasoning.

## Why this explains the model's abilities

If you accept next-token prediction as the actual mechanism, a lot of
LLM behavior stops looking mysterious. The model follows instructions well
because its training data is full of examples of instructions being
followed, so "an instruction followed by compliant text" is a pattern it has
seen enormously often and learned to continue. It can write in a specific
style because style is itself a statistical pattern in text, and continuing
that pattern is exactly what the model is built to do. Even behaviors that
look like multi-step reasoning are, mechanically, the model predicting the
kind of text that follows "let's work through this step by step" — which
happens to often take the form of genuine, useful intermediate steps,
because that's what such text looks like in the model's training data.

This also sets up why context matters so much, a thread the next lesson
picks up directly: since every prediction is conditioned on everything
that came before it in the sequence, anything you want the model to use —
an instruction, a fact, an example — has to actually be present in that
sequence. The model has no separate memory to draw on mid-generation.

## Why this explains the model's failures

The same mechanism that produces fluent, often-correct text has no built-in
mechanism for distinguishing "this continuation is factually true" from
"this continuation is a statistically plausible thing to say here." The
model is optimizing for *plausible continuation*, not *verified truth*.
Most of the time these line up, because plausible text about real things
tends to be true — but when they diverge, the model has no internal alarm
bell. It will produce a confident, fluent, wrong answer with exactly the
same mechanism it uses to produce a confident, fluent, correct one. That's
the seed of hallucination, which gets its own lesson later in this roadmap,
but the root cause is visible right here: there is no truth-checking step
in "predict the next plausible token."

It's also why models can be inconsistent across runs on the exact same
prompt. Generation involves sampling from a probability distribution rather
than always taking the single highest-probability token — a setting called
temperature controls how much randomness that sampling has, and you'll see
the mechanics of that in the next group of lessons. Even at low randomness,
small differences accumulate: the model samples a slightly less likely
token early in the response, and every token after that is now conditioned
on a different sequence than it would have been otherwise.

## The mental model to carry forward

Treat an LLM as an exceptionally capable pattern-continuation engine, not as
an agent that understands your request the way a person would. It has no
goals, no persistent memory between separate calls, and no mechanism to
check its own output against reality unless you build that checking into
your process. Everything useful you can do with prompt engineering follows
from working with this mechanism rather than against it: you supply the
context it needs because it has no other source of context, you specify the
format you want because it will otherwise continue in whatever format is
statistically typical, and you build in verification because the model
itself has none. The rest of this roadmap is a systematic way of doing
exactly that.
