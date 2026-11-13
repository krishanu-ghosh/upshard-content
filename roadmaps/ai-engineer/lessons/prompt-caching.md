---
title: Prompt caching
description: How a cached prefix removes repeated prefill work, how to arrange a prompt so the cache actually hits, and what it saves.
roadmap: ai-engineer
node: prompt-caching
group: prompting
video:
  provider: youtube
  id: "Q_U4pRggC5I"
  start: 0
  end: null
creator:
  name: Efficient NLP
  channel_url: https://www.youtube.com/@EfficientNLP
duration_seconds: 531
level: intermediate
prerequisites: [context-engineering]
takeaways:
  - Caching stores the key-value tensors computed during prefill, so a repeated prefix is loaded rather than recomputed.
  - The cache matches on an exact prefix from the first token, so anything variable must come after everything stable.
  - The saving grows with how much of the prompt is shared, which makes it largest exactly where costs hurt most — long system prompts and agent loops.
check_questions:
  - q: Why must a timestamp or user id go at the end of a prompt rather than the start?
    a: Matching is on an exact prefix from the first token. A value that changes per request invalidates everything after it, so putting it first means nothing is ever cached.
  - q: Why do agent loops benefit from caching more than single-turn chat?
    a: Each iteration re-sends the whole accumulated history, so the shared prefix grows with every step. Without caching an agent pays repeatedly to prefill the same tokens.
deeper:
  - label: "Hugging Face — Prompt caching explained, with agent cost examples"
    url: https://www.youtube.com/watch?v=SkM4k4SKvCM
  - label: "IBM Technology — What is prompt caching?"
    url: https://www.youtube.com/watch?v=u57EnkQaUTY
related_formulas: [kv-cache-size, memory-bound-token-latency]
---

## The problem: paying for the same work twice

Serving a model call has two phases. **Prefill** reads the entire input in one
parallel pass, building the key and value tensors for every token at every
attention layer — the same KV cache described in the
[context window](/roadmaps/ai-engineer/the-context-window) lesson. **Decode**
then generates output tokens one at a time, each step reading what prefill
built. Prefill is the expensive part, and it scales with input length. Now
notice something: if your next request opens with the exact same two thousand
tokens as the last one — the same system prompt, the same tool definitions,
the same long reference document — the provider redoes all of that prefill
work from scratch, even though the answer would come out identical every
time. That waste is the whole problem caching exists to fix.

## What gets reused

Think of it like a kitchen that preps a big pot of stock once at the start of
service, instead of making fresh stock for every single order. Caching keeps
the computed key and value tensors for a prefix around between calls, keyed to
that exact sequence of tokens. When a new request opens with a prefix that
matches one already cached, the provider skips redoing that part of prefill
and just reuses the stored tensors, picking up fresh computation only from the
first token that differs. The cached portion bills at a small fraction of its
normal per-token price and adds almost no latency, because the expensive
parallel work behind it was already paid for once. Nothing about what the
model can attend to or how it reasons changes — this is purely about how much
redundant computation the provider has to redo.

## Why the match has to be exact, from token one

The cache keys on an identical prefix starting at position zero — not on
"close enough," not on semantic similarity. One differing token anywhere in
the prefix, and everything from that position onward falls out of cache,
because each token's key and value tensors depend on every token before it
through attention. That single fact decides how you should lay out a prompt:
put the most stable material first — system instructions, tool definitions,
long reference documents, since these barely change call to call — then
retrieved documents, and only at the very end anything request-specific, like
a timestamp or a session ID. Put that timestamp near the top out of habit and
you've invalidated the cache on every single call, since no two requests ever
share that value.

## Doing the arithmetic

Say a system prompt plus tool definitions run three thousand tokens, and an
agent loop reuses them across twenty iterations per task. Without caching,
that's three thousand tokens of prefill work repeated twenty times over — sixty
thousand tokens of computation spent on content that never changed. With
caching, only the first call pays full price for that chunk; the other
nineteen pay the steeply discounted cached rate, with full price landing only
on the new content appended at the end. The saving scales with how much of
the prompt is shared and how often it repeats, which is exactly why agent
loops — where most of the prompt is identical turn after turn — gain far more
from this than a single-turn chat ever would.

## What differs by provider

Providers disagree on the minimum prefix length that qualifies for caching,
on how long a cached entry survives before it's evicted, and on whether
caching happens automatically or needs an explicit marker in the request
showing where the cacheable prefix ends. Check the current specifics directly
rather than assuming last year's numbers still hold. What doesn't change
across providers is the underlying shape of the trick — stable content first,
variable content last — and it stays one of the cheapest, biggest cost
reductions available to most LLM applications, tying straight into the
broader economics in [cost and latency](/roadmaps/ai-engineer/cost-and-latency).
