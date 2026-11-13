---
title: KV cache mechanics
description: What is actually stored in the KV cache, why it exists, and how its size scales with context length, batch size and model size — from the serving engine's side.
roadmap: inference-engineering
node: kv-cache-mechanics
group: inside-the-forward-pass
video:
  provider: youtube
  id: "hafEw3bEu8E"
  start: 0
  end: null
creator:
  name: Ready Tensor
  channel_url: https://www.youtube.com/@ready_tensor
duration_seconds: 727
level: advanced
prerequisites: [prefill-and-decode]
takeaways:
  - The KV cache stores the key and value tensors computed for every past token, at every attention layer and head, so decode never has to recompute attention over tokens it has already processed.
  - Without the cache, every decode step would repeat the full attention computation over the entire sequence so far, turning a cheap sequential step into an expensive, repeated one.
  - Cache size scales directly with context length, batch size and model size, which makes it the dominant consumer of GPU memory during serving — often larger than the model's own weights at long context or high concurrency.
check_questions:
  - q: What specifically does the KV cache store, and why does that make decode cheap?
    a: It stores the key and value vectors computed during attention for every past token, at every layer and every attention head. Because those are cached rather than recomputed, each new decode step only has to compute attention for the one new token against the cached keys and values, not redo the whole sequence.
  - q: Why can the KV cache become larger than the model's own weights?
    a: Cache size grows with context length and batch size — more tokens in context, or more concurrent requests, means more cached key/value pairs stored simultaneously. At long context lengths or high concurrency, that total can exceed the fixed size of the model's weights, which do not grow with either of those.
deeper:
  - label: "NVIDIA Developer — Distributed Inference 101: Managing KV Cache"
    url: https://www.youtube.com/watch?v=SeImiPDVMCw
related_formulas: [kv-cache-size]
---

## Why caching exists at all

As the previous lesson covered, each decode step needs attention computed
between the new token and every token that came before it. Recomputing that
attention from scratch at every single step — redoing the work for every
earlier token, every time a new one is generated — would make decode
dramatically more expensive than it needs to be, since the vast majority of
that computation produces exactly the same result it produced the step
before. The **KV cache** avoids this by storing the result once and reusing
it.

## What is actually stored

At every attention layer, for every attention head, a token's contribution
to attention is captured in two vectors: a **key** and a **value**. Once a
token has been processed, its key and value vectors are fixed and will never
change — they depend only on that token and the ones before it, not on
anything generated afterward. The KV cache stores exactly these vectors, for
every past token, at every layer and head, so that a new decode step only
has to compute the key and value for the one new token and compare it
against everything already cached, rather than reprocessing the whole
sequence.

## Why the cache is often the biggest thing in memory

Cache size scales with three things at once: context length (more tokens
means more cached key/value pairs per request), batch size (more concurrent
requests means more sets of cached pairs held simultaneously), and model size
(more layers and heads means more key/value vectors per token). See
[KV cache size](/formulas/kv-cache-size) for the exact arithmetic. Unlike the
model's own weights, which occupy a fixed amount of memory regardless of
traffic, the cache's footprint grows directly with how much context and how
many concurrent users a server is actually handling — which is why, at long
context lengths or high concurrency, the cache routinely becomes the largest
single consumer of GPU memory, sometimes dwarfing the weights it is serving
alongside.

## A concrete sense of the memory involved

A model with several dozen layers and tens of attention heads per layer, each
storing a key and value vector of a few thousand numbers for every single
token in context, adds up quickly. A single request holding a context of a
few thousand tokens can occupy several gigabytes of cache on its own, and
that figure multiplies directly with however many such requests a server is
holding concurrently. This is why context length and concurrency are not
independent knobs a serving system can tune freely — doubling either one
roughly doubles the cache memory required, and a server that comfortably
handles a hundred short conversations at once may struggle badly the moment
even a handful of those conversations grow into long ones.

## Why this matters for everything downstream

Managing this memory efficiently, rather than wasting it, is the direct
motivation behind [PagedAttention](/roadmaps/inference-engineering/flashattention-and-pagedattention), which manages the cache the way an operating
system manages virtual memory to avoid fragmentation and waste. Sharing a
cached prefix across multiple requests that share a common prompt start is
the whole idea behind
[prefix caching](/roadmaps/inference-engineering/prefix-caching). And the
cache's sheer size at scale is a direct input into
[choosing hardware](/roadmaps/inference-engineering/choosing-hardware) —
sizing a GPU for a workload means budgeting for the cache's likely footprint,
not just the model's weights.
