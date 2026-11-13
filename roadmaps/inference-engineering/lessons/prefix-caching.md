---
title: Prefix caching
description: Reusing the KV cache computed for a shared prompt prefix across multiple requests, at the serving-engine level.
roadmap: inference-engineering
node: prefix-caching
group: speeding-up-inference
video:
  provider: youtube
  id: "mhZahVv3A3I"
  start: 0
  end: null
creator:
  name: Standarity
  channel_url: https://www.youtube.com/@Standarity
duration_seconds: 283
level: advanced
prerequisites: [flashattention-and-pagedattention, continuous-batching]
takeaways:
  - When multiple requests share an identical prompt prefix — a common system prompt, a shared set of instructions — the KV cache for that shared portion can be computed once and reused across every request that shares it.
  - A radix tree indexing cached prefixes lets a serving engine quickly find the longest matching prefix already cached for a new incoming request, rather than checking every cached prefix individually.
  - This is the serving-engine-level counterpart to application-level prompt caching — the same underlying idea of not recomputing a shared prefix, implemented inside the engine rather than exposed as an API feature.
check_questions:
  - q: Why does prefix caching help when many requests share an identical system prompt?
    a: Computing the KV cache for that shared prefix is identical work every single time it recurs, since the prefix itself never changes. Caching it once and reusing it across every request that shares it skips recomputing that identical work from scratch each time.
  - q: What does a radix tree let the serving engine do that checking every cached prefix individually would not?
    a: It lets the engine quickly find the longest prefix of a new request that already exists somewhere in the cache, by walking down a tree structured around shared prefixes, rather than comparing the new request against every previously cached prefix one at a time.
deeper:
  - label: "Bob Ren — Radix attention in SGLang"
    url: https://www.youtube.com/watch?v=4ON42F1CyHI
related_formulas: [kv-cache-size]
---

## The same prefix, computed over and over

Many real workloads send requests that share a large, identical prefix —
every call to a particular feature might start with the same lengthy system
prompt, or a conversation's growing history is sent in full on every turn
with only the latest message actually new. As covered in
[KV cache mechanics](/roadmaps/inference-engineering/kv-cache-mechanics),
computing the KV cache for a prefix is real, non-trivial work, and if that
exact same prefix recurs across many requests, recomputing it identically
every single time is pure waste — the result would be exactly the same as
last time, since nothing about that shared prefix has changed.

## What prefix caching actually does

**Prefix caching** stores the KV cache for a prefix once it has been
computed, keyed by the prefix's content, and reuses that stored cache for any
subsequent request that shares the same beginning. Only the genuinely new
portion of a request — whatever comes after the point where it diverges from
anything previously seen — actually needs fresh computation. For a system
prompt shared across every single call to a feature, this can mean the
large, fixed portion of every request's prefill cost is paid only once,
rather than on every call.

## Why a tree, rather than a simple lookup

A naive implementation might check a new request's prefix against every
previously cached prefix individually, which becomes slow as the number of
cached prefixes grows. A **radix tree** instead organises cached prefixes
hierarchically, by their shared beginnings, so finding the longest matching
prefix for a new request means walking down the tree following whichever
branch matches, rather than comparing against every stored prefix one at a
time. This is what makes prefix caching practical at the scale of a real
production system serving many different prefixes simultaneously, rather
than only working well for the simple case of one single shared system
prompt.

## Eviction: the cache cannot grow forever

A cache that only ever grows would eventually consume all available GPU
memory, so a serving engine needs a policy for what to evict when space runs
short. The usual approach is some form of least-recently-used eviction,
removing the cached prefixes that have gone longest without being reused,
on the reasoning that a prefix nobody has asked for recently is the least
likely to be asked for again soon. This interacts directly with
[PagedAttention](/roadmaps/inference-engineering/flashattention-and-pagedattention)'s block-based memory management — individual pages belonging to a
prefix can be evicted and reassigned to new requests as needed, rather than
having to evict or keep an entire prefix's cache as one indivisible unit,
which gives the engine much finer-grained control over exactly how much
memory stays reserved for caching versus how much is available for
processing new, uncached requests.

## The serving-engine mirror of an application-level feature

This is mechanically the same underlying idea as the prompt caching covered
in Upshard's AI Engineer roadmap, viewed from the opposite side: that lesson
covers prompt caching as something an application developer arranges for by
structuring a prompt's stable content first. This lesson covers the same
avoided-recomputation idea from inside the serving engine itself, which is
what actually makes that application-level feature possible at all — the
engine has to be capable of recognising and reusing a shared prefix before
any application developer can rely on it happening.
