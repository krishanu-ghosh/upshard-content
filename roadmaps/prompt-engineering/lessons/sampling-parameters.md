---
title: Sampling parameters
description: An LLM never deterministically picks the next word — sampling parameters control how it rolls the dice over a probability distribution.
roadmap: prompt-engineering
node: sampling-parameters
group: model-fundamentals
video:
  provider: youtube
  id: "MkaazQttbpc"
  start: 0
  end: null
creator:
  name: Gary Explains
  channel_url: https://www.youtube.com/@GaryExplains
duration_seconds: 891
level: beginner
prerequisites: []
takeaways:
  - A model's forward pass produces a full probability distribution over every possible next token, not a single predicted token.
  - Sampling parameters — temperature, top-k, and top-p — control how a token gets drawn from that distribution, and changing them changes the model's behavior without changing the prompt at all.
  - Lower-randomness settings suit factual and structured tasks; higher-randomness settings suit creative and exploratory ones.
check_questions:
  - q: If sampling parameters are separate from the prompt, why does a prompt-engineering roadmap cover them?
    a: Because the same prompt produces different outputs depending on how a token gets picked from the model's output distribution. Writing a correct prompt and then running it with parameters that fight your intent — high temperature on a task that needs one exact answer, for instance — wastes the prompt's precision.
  - q: Two API calls use the exact same prompt and model, but return different text both times. Is the model behaving incorrectly?
    a: No. Unless sampling is turned off (e.g. temperature set to 0, and even then not always perfectly on every provider), the model is drawing from a probability distribution each time rather than returning a single fixed answer, so some variation between runs is expected behavior, not a bug.
deeper:
  - label: "DataMListic — LLM Prompt Engineering with Random Sampling: Temperature, Top-k, Top-p"
    url: https://www.youtube.com/watch?v=-BBulGM6xF0
related_formulas: [softmax, temperature-sampling, top-p-sampling]
---

## A model outputs a distribution, not an answer

It is tempting to picture a language model as a lookup machine: feed it a
prompt, it computes "the" next word. That is not how generation works. For
every position in the output, the model's final layer produces one real
number — a logit — per token in its vocabulary, tens of thousands of numbers
scoring how plausible each possible next token is given everything so far.
Those logits get passed through a [softmax](/formulas/softmax) function,
which converts them into a probability distribution: every token gets a
probability between 0 and 1, and the whole set sums to 1. Generating text
means repeatedly picking one token from that distribution, appending it, and
running the model again for the next position. Sampling parameters are the
rules for how that pick gets made — and because the same distribution can
produce very different text depending on how you sample from it, they
matter as much as the prompt that produced the distribution in the first
place.

## Why not just always pick the top token?

The simplest possible strategy — always take the single highest-probability
token, called greedy decoding — sounds like it should give the "best"
output, but in practice it produces text that is often repetitive and
strangely generic. Language has many locally reasonable continuations at
almost every point, and always taking the top-ranked one steers the model
into well-worn phrasing and, in longer generations, into loops where it
repeats the same clause because that was marginally the highest-scoring
choice at each step. Introducing controlled randomness — sampling rather
than always taking the top token — breaks that pattern and generally
produces more natural, more varied text, at the cost of occasionally
sampling a token that is a worse choice than the top one would have been.
Sampling parameters are how you tune that trade-off.

## The three knobs

Three parameters are standard across essentially every LLM API, and each
one adjusts the trade-off between consistency and variety in a different
way. **Temperature** reshapes the probability distribution itself before
sampling — low temperature sharpens it toward the highest-probability
tokens (closer to greedy decoding), high temperature flattens it, giving
lower-probability tokens a real chance of being picked. It is covered in
depth in [temperature and randomness](/roadmaps/prompt-engineering/temperature-and-randomness).
**Top-k** restricts sampling to only the k highest-probability tokens at
each step, discarding the long tail entirely regardless of how flat or
peaked the distribution is. **Top-p** (nucleus sampling) instead keeps the
smallest set of top tokens whose cumulative probability exceeds p, so the
number of candidates it keeps shrinks automatically when the model is
confident and grows automatically when it is uncertain. Both top-k and
top-p are covered together, with the trade-offs between them, in
[top-k and top-p](/roadmaps/prompt-engineering/top-k-and-top-p). All three
typically apply together: temperature reshapes the distribution, then top-k
or top-p truncates it before a token is drawn.

## Matching randomness to the task

The right setting depends entirely on what you are asking the model to do.
Tasks with one correct answer — extracting a value from text, classifying
a ticket into a fixed category, generating code that must compile, answering
a factual question — want low randomness: you want the model's most
confident answer nearly every time, and variation mostly just introduces
errors. Tasks with many acceptable answers — brainstorming names, writing
marketing copy, generating varied synthetic examples — benefit from higher
randomness, because the value of the output is partly in its diversity, and
a model locked near-greedy will give you five near-identical variations of
the same idea. A useful habit is deciding, before you write a prompt, which
category the task falls into, rather than leaving sampling parameters at
whatever default the API happens to ship with.

## Parameters do not fix a bad prompt

It is worth being clear about what sampling parameters cannot do. They
change how a token gets picked from a distribution the model already
computed — they have no influence on what that distribution contains.
If a prompt is ambiguous, missing necessary context, or asks for something
the model is not capable of, no combination of temperature, top-k, or top-p
will produce a correct answer; at best, a lower-randomness setting will make
the same wrong answer come back more consistently. Sampling parameters are a
tool for controlling the *style* and *variability* of correct or plausible
output, not a substitute for a well-constructed prompt, which is why this
roadmap treats them as a dial you turn after the prompt itself is right,
never as the first thing to reach for when output quality disappoints.
