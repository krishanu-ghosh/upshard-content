---
title: Time to first token
description: How long before the first output token appears, why prefill dominates it, and why it is the latency users actually feel.
roadmap: inference-engineering
node: time-to-first-token
group: latency-and-throughput
video:
  provider: youtube
  id: "9tvJ_GYJA-o"
  start: 0
  end: 1241
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 2019
level: advanced
prerequisites: [the-inference-stack]
takeaways:
  - Time to first token (TTFT) measures how long a user waits before anything appears on screen, and it is dominated by the prefill phase — processing the entire prompt before generation can begin.
  - TTFT grows with prompt length, because prefill has to process every input token before producing the first output token, which is why a long system prompt or large retrieved context directly costs latency.
  - TTFT is the number a user notices most, because it governs the length of the silent wait before a streaming interface shows anything at all — everything after that first token is perceived differently.
check_questions:
  - q: Why does a longer prompt directly increase time to first token?
    a: The entire prompt has to be processed in the prefill phase before the first output token can be produced, and prefill's cost scales with how much input there is to process. More tokens in means more work before anything comes out.
  - q: Why does TTFT matter more to perceived speed than total response time?
    a: A streaming interface shows nothing until the first token arrives, so TTFT is the length of the blank, silent wait a user actually experiences. Once tokens start streaming, the perception of speed changes even if the total generation still takes a while.
deeper: []
related_formulas: []
---

## The number a user actually waits on

Time to first token is exactly what it sounds like: the elapsed time between
sending a request and seeing the first piece of the response appear. In a
streaming chat interface, this is the length of the blank, silent pause before
anything shows up at all — and it is the single latency number most directly
tied to how fast a product *feels*, independent of how long the full response
eventually takes to finish streaming.

## Why prefill dominates it

Generating a response has two phases, covered properly in the
[next lesson](/roadmaps/inference-engineering/prefill-and-decode): prefill,
which processes the entire input prompt in one parallel pass, and decode,
which produces output tokens one at a time afterward. The first output token
cannot appear until prefill has finished processing the whole prompt — there
is no way to start generating before the model has seen everything it was
given. This means TTFT is, to a very good approximation, the time prefill
takes, and prefill's cost scales directly with how many input tokens there
are to process.

## Why this means your prompt length is a latency decision

A system prompt padded with unnecessary instructions, or a retrieval step
that returns more context than the question actually needs, is not merely a
cost decision — it is a direct tax on every single request's TTFT, because
every one of those extra tokens has to be processed in prefill before the
user sees anything. This is the same economic pressure that makes
[prompt caching](/roadmaps/inference-engineering) worth having at the serving
level: a cached prefix skips re-processing exactly the part of prefill that
does not change between requests, which is a direct TTFT win and not merely a
cost one.

## A worked sense of the numbers

Prefill cost scales with prompt length in a way that is worth seeing
concretely. A short, ten-token question might take a few tens of
milliseconds to process on a well-provisioned GPU. A prompt padded with a
lengthy system message, several retrieved documents and a long conversation
history — easily several thousand tokens — can take an order of magnitude
longer to prefill, simply because there is that much more to process before
the first output token can appear. This is why a product with a verbose,
unoptimised system prompt can feel noticeably slower to start responding than
one with a lean prompt, even when both are calling the identical model on
identical hardware — the difference is entirely in how much prefill work each
one demands before anything becomes visible.

## TTFT is not the whole latency story

A fast TTFT with slow subsequent generation still leaves a user waiting a
long time for the complete answer — TTFT is necessary for a product to feel
responsive, not sufficient for it to actually finish quickly. The next two
lessons, on [throughput](/roadmaps/inference-engineering/tokens-per-second-and-throughput) and the
[tradeoff between latency and throughput](/roadmaps/inference-engineering/latency-vs-throughput), cover the rest of what determines how fast a
response actually completes once it has started.
