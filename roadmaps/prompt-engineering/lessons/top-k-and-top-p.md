---
title: Top-k and top-p
description: Two ways to truncate a model's output distribution before sampling — a fixed-size shortlist against one that adapts to the model's confidence.
roadmap: prompt-engineering
node: top-k-and-top-p
group: model-fundamentals
video:
  provider: youtube
  id: "QgneZX57bsI"
  start: 0
  end: null
creator:
  name: PM Expert
  channel_url: https://www.youtube.com/@pankajsh10
duration_seconds: 119
level: beginner
prerequisites: [sampling-parameters]
takeaways:
  - Top-k keeps only the k highest-probability tokens at every step, regardless of how confident or uncertain the model actually is.
  - Top-p (nucleus sampling) keeps the smallest set of tokens whose cumulative probability passes a threshold p, so the candidate pool shrinks automatically when the model is confident and grows when it isn't.
  - Top-p is the more commonly used default today because it adapts to the shape of the distribution, where top-k's fixed count is either too loose or too strict depending on the token.
check_questions:
  - q: Why can a fixed top-k value be too restrictive at some points in a generation and too loose at others?
    a: Because the model's confidence varies token by token. When the model is very confident (most of the probability mass sits on one or two tokens), a top-k of 40 still includes many implausible tail tokens that shouldn't realistically be candidates. When the model is uncertain (probability spread thinly across hundreds of plausible tokens), the same top-k of 40 may cut off reasonable options purely because they didn't make an arbitrary fixed cutoff.
  - q: A model is choosing the next word after "The capital of France is", where one token (Paris) holds 97% of the probability mass. With top-p set to 0.9, roughly how many tokens are available to sample from, and why?
    a: Essentially just one — Paris alone already exceeds the 0.9 cumulative-probability threshold, so nucleus sampling includes only that token (or very close to it). This is exactly the adaptive behavior top-p is designed for — when the model is this confident, it narrows almost to greedy decoding automatically, without needing top-k's fixed count to happen to be small enough.
deeper: []
related_formulas: [top-p-sampling]
---

## Truncating the distribution before you sample

[Temperature](/roadmaps/prompt-engineering/temperature-and-randomness)
reshapes a model's output distribution, but it never removes any token from
consideration — given enough samples, even a wildly implausible token has
some nonzero chance of being drawn, however flat the distribution gets.
Top-k and top-p are the two standard ways to cut that long tail off
entirely before a token gets drawn, so that implausible tokens simply
cannot be selected no matter how unlucky the random draw is. Both run after
temperature has already reshaped the distribution and before the actual
sampling step picks one token from what remains.

## Top-k: a fixed-size shortlist

Top-k sampling is the simpler of the two: sort every possible next token by
probability, keep only the k highest-ranked ones, discard everything else,
and renormalize the remaining probabilities so they sum back to 1 before
sampling. A top-k of 40 means only the 40 most likely tokens at that
position are ever eligible, regardless of what their actual probabilities
look like. This is easy to reason about and cheap to compute, and it was
the standard truncation method before nucleus sampling was introduced. Its
weakness is that the number 40 means something very different depending on
how peaked or flat the underlying distribution is at that specific point in
generation.

## Top-p: a cutoff that adapts to confidence

Top-p sampling, also called nucleus sampling, works differently: rather
than fixing how many tokens to keep, it fixes how much cumulative
probability to keep (see [top-p sampling](/formulas/top-p-sampling)). Sort
tokens by probability, then add them to the candidate set one at a time,
highest first, until their combined probability crosses the threshold p —
commonly 0.9 or 0.95. When the model is very confident about the next
token (most probability concentrated on one or two options), the nucleus
might contain only one or two tokens even with p set high, because those
few tokens alone already cross the threshold. When the model is genuinely
uncertain (probability spread thin across many plausible continuations),
the nucleus automatically grows to include dozens of tokens, because it
takes more of them to reach the same cumulative threshold. The candidate
pool's size responds to the shape of the distribution at each individual
step, which is exactly what a fixed top-k cannot do.

## Why top-p is the more common default

This adaptiveness is the practical reason most APIs default to top-p, or
offer both and recommend top-p for general use. A fixed top-k has to be
tuned as a compromise: set it low, and you risk excluding reasonable
options at exactly the moments the model is uncertain and needed more
candidates, not fewer; set it high, and you risk including genuinely poor
options at the moments the model was confident and a tight shortlist would
have served you better. Top-p sidesteps that compromise by construction,
widening and narrowing itself automatically. This doesn't make top-k
useless — it's simpler, slightly cheaper to compute, and some practitioners
still prefer the predictability of a fixed candidate count — but it explains
why documentation and defaults across most major providers lean toward
top-p as the general-purpose choice.

## None of this works alone

Top-k and top-p are not usually used in isolation, and they are not usually
used instead of temperature — the standard pattern applies temperature
first to reshape the distribution, then top-p or top-k to cut its tail, then
samples from what's left. It's also common for a model provider to apply
both top-k and top-p together as a belt-and-suspenders truncation, with
top-k serving as a hard upper bound on candidate count even when top-p alone
would have kept more tokens. Practically, when a model's output already
reads as reasonably on-target but occasionally drifts into clearly
nonsensical territory, tightening top-p (lowering it toward 0.85 or so) is
usually a better first fix than lowering temperature, since it directly
removes the implausible tail rather than globally flattening or sharpening
the whole distribution the way temperature does.
