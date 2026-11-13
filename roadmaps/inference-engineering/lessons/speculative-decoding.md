---
title: Speculative decoding
description: A small, fast draft model proposes several tokens at once, verified in parallel by the full model — speeding up generation without changing its output.
roadmap: inference-engineering
node: speculative-decoding
group: speeding-up-inference
video:
  provider: youtube
  id: "1XDi8_VPCDU"
  start: 0
  end: null
creator:
  name: EleutherAI
  channel_url: https://www.youtube.com/@Eleuther_AI
duration_seconds: 5762
level: advanced
prerequisites: [quantization]
takeaways:
  - Speculative decoding uses a small, fast draft model to propose several candidate tokens at once, which the full target model then verifies in a single parallel pass rather than generating each one sequentially itself.
  - Verification is exact, not approximate — any proposed token the target model would not actually have produced is rejected and regenerated correctly, so the final output is identical in distribution to normal decoding.
  - The speedup comes from converting several of decode's expensive, memory-bound sequential steps into one parallel, more compute-like verification pass, which is cheaper when the draft model guesses correctly often enough.
check_questions:
  - q: Why does speculative decoding not change the quality or distribution of the final output?
    a: The target model verifies every proposed token exactly against what it would have generated on its own, and rejects and correctly regenerates any token that does not match. The mechanism only changes how the computation is scheduled, not what the model is ultimately allowed to say.
  - q: Why can verifying several proposed tokens at once be cheaper than generating them one at a time?
    a: Verification can be done in a single parallel pass over all the proposed tokens, similar in character to prefill, rather than requiring a separate sequential, memory-bound decode step for each one. As long as the draft model's guesses are often correct, this trades several expensive sequential steps for one cheaper parallel one.
deeper:
  - label: "Hugging Face — Local AI 202: Massive Speed-ups with Speculative Decoding"
    url: https://www.youtube.com/watch?v=H4azmWcqO5g
related_formulas: []
---

## Guess several steps ahead, then check all at once

Ordinary decode generates one token at a time, each step a separate,
sequential, memory-bound pass through the full model — covered in
[prefill and decode](/roadmaps/inference-engineering/prefill-and-decode) and
[memory-bound vs compute-bound](/roadmaps/inference-engineering/memory-bound-vs-compute-bound). **Speculative decoding** changes the schedule of this work
without changing what the model is allowed to say: a small, fast **draft
model** proposes several candidate next tokens at once, guessing ahead of
where the full **target model** has actually computed to. The target model
then checks all of those proposed tokens in a single parallel pass, similar
in character to prefill, rather than generating each one through its own
separate sequential step.

## Why verification is exact, not a shortcut on quality

This is not an approximation technique. For each proposed token, the target
model computes what it would actually have generated at that position on its
own, and compares it against the draft model's guess. A correct guess is
accepted and the computation moves on to checking the next one; an incorrect
guess is rejected, and the target model's own correct token is used instead,
with every proposal after that point discarded and the process restarting
from there. The final sequence of tokens produced is identical, in
distribution, to what ordinary step-by-step decoding would have produced —
the technique only changes how the computation is scheduled, never what the
model is permitted to output.

## Why this actually saves time

The saving comes from exploiting the exact asymmetry covered in
[memory-bound vs compute-bound](/roadmaps/inference-engineering/memory-bound-vs-compute-bound): a sequential decode step pays the full cost of moving the
target model's weights through memory for just one token's worth of useful
output. Verifying several proposed tokens at once pays a similar cost in
memory movement but produces several tokens' worth of output if the draft
model's guesses hold up, converting several expensive sequential passes into
one pass that does more useful work per unit of memory traffic. This only
pays off when the draft model is cheap enough that proposing tokens barely
costs anything and accurate enough that most of its guesses survive
verification — a draft model that guesses poorly gains nothing, since every
rejected proposal is wasted computation on top of what ordinary decoding
would have cost anyway.

## A worked round

Say the draft model proposes 4 tokens per round, and historically about 80%
of its guesses survive verification. On a typical round the target model
accepts the first few proposals, hits one it disagrees with, substitutes its
own correct token at that position, and discards whatever the draft model
guessed after it — since those later guesses were conditioned on a token
that never actually got produced. A round that accepts 3 of 4 proposals
still yields 4 verified tokens (3 accepted plus the target's own correction)
from a single parallel pass, instead of 4 separate sequential decode steps.
The draft model's own forward passes are cheap, since it is small, so even
when a round only partially succeeds the trade is almost always still worth
making.

## Named variants worth recognising

**Medusa** and **EAGLE** are specific, more sophisticated approaches to
generating the draft proposals, each improving on simpler draft-model setups
in different ways — EAGLE in particular has gone through several iterations
(EAGLE, EAGLE-2, EAGLE-3) each refining how the draft predictions are
produced. Recognising these names is useful context for keeping up with this
fast-moving area; the underlying draft-then-verify mechanism covered above is
what every one of them builds on.
