---
title: FlashAttention and PagedAttention
description: FlashAttention restructures attention to avoid wasted memory traffic; PagedAttention manages the KV cache like virtual memory to reduce fragmentation.
roadmap: inference-engineering
node: flashattention-and-pagedattention
group: speeding-up-inference
video:
  provider: youtube
  id: "6uPnLkCiy5g"
  start: 0
  end: null
creator:
  name: Tales Of Tensors
  channel_url: https://www.youtube.com/@TalesOfTensors
duration_seconds: 413
level: advanced
prerequisites: [kernel-fusion, kv-cache-mechanics]
takeaways:
  - PagedAttention manages the KV cache the way an operating system manages virtual memory — in fixed-size blocks that can be allocated non-contiguously, which eliminates the waste of reserving one large contiguous region per request.
  - FlashAttention restructures the attention computation itself to avoid ever materializing the full attention matrix in slow memory, keeping intermediate results in fast on-chip memory throughout.
  - Both are exact optimizations, not approximations — they produce the same output as the naive implementation, just computed with far less wasted memory traffic.
check_questions:
  - q: What specifically does PagedAttention borrow from how operating systems manage memory?
    a: It allocates the KV cache in fixed-size blocks, similar to memory pages, that do not need to sit contiguously in memory and can be assigned to a request as needed rather than reserving one large contiguous region upfront sized for the request's worst-case length.
  - q: What problem does reserving one large contiguous KV cache region per request actually cause?
    a: Since a request's final length is not known in advance, a naive implementation has to reserve memory for the maximum possible length upfront, which wastes most of that reservation for requests that turn out to be much shorter — and fragments memory so that even unused reserved space cannot be given to other requests.
deeper:
  - label: "Data Science Gems — Flash Attention"
    url: https://www.youtube.com/watch?v=1RaIS98jj1Q
related_formulas: [kv-cache-size, scaled-dot-product-attention]
---

## Two different wastes, two different fixes

This lesson covers two of the most influential systems optimisations in
modern LLM serving, each targeting a different kind of waste.
**PagedAttention** addresses waste in how the KV cache's memory is
allocated. **FlashAttention** addresses waste in how the attention
computation itself moves data through memory. Both are **exact**
optimisations — neither changes the model's output at all, only how
efficiently that identical output is computed.

## PagedAttention: memory management borrowed from operating systems

A naive KV cache implementation has to reserve memory for each request
before knowing how long that request will actually run — since a request's
final length is not known in advance, the naive approach reserves enough
contiguous memory for the worst case. This wastes most of that reservation
for the many requests that turn out to be much shorter, and the fragmented
leftover space often cannot be reused for anything else, per
[KV cache size](/formulas/kv-cache-size). **PagedAttention** fixes this by
borrowing a well-established idea from operating systems: allocate the cache
in small, fixed-size blocks — pages — that do not need to sit next to each
other in memory and can be assigned to a request incrementally, as it
actually generates tokens, rather than reserved all at once upfront. This
eliminates both the waste of over-reservation and the fragmentation that
comes with it.

## What a block table actually looks like

PagedAttention keeps, for every request, a small **block table**: a list
mapping each logical position in that request's sequence to a physical page
somewhere in GPU memory. Appending a new token only ever needs one more
entry in this list and, if the current page is full, one more physical page —
never a search for a large contiguous span of free memory. This is exactly
why fragmentation stops being a problem: a page freed by one finished request
can be handed to any other request's block table immediately, regardless of
where in physical memory it happens to sit, because nothing downstream
depends on a request's pages being next to each other.

## FlashAttention: keeping the computation on-chip

Computing attention naively involves materialising a full matrix of
attention scores — one entry per pair of positions in the sequence — in the
GPU's main memory, then reading that matrix back in for the next step of the
computation. For anything beyond a short sequence this matrix is large, and
writing and then re-reading it is exactly the kind of unnecessary memory
traffic [kernel fusion](/roadmaps/inference-engineering/kernel-fusion)
exists to eliminate. **FlashAttention** restructures the computation to avoid
ever fully materialising this matrix in slow memory at all — processing the
attention computation in smaller tiles that fit in fast, on-chip memory
throughout, and only writing the final result back out. The output is
identical to naive attention; what changes is that far less data makes the
expensive round trip to and from main memory along the way.

## Why both matter together

PagedAttention and FlashAttention target different parts of the same
problem — memory management for the cache, and data movement within the
attention computation itself — and a mature serving engine implements both
rather than choosing between them. Together they are a large part of why a
properly built inference engine, covered in
[inference engines](/roadmaps/inference-engineering/inference-engines),
outperforms a naive implementation so substantially: neither technique
changes what the model computes, only how much wasted memory traffic that
computation requires to produce the identical result.
