---
title: Kernel fusion
description: Combining multiple small GPU operations into one kernel to avoid repeatedly reading and writing memory between them.
roadmap: inference-engineering
node: kernel-fusion
group: serving-software
video:
  provider: youtube
  id: "m6BSREnQ84U"
  start: 0
  end: null
creator:
  name: GPU MODE
  channel_url: https://www.youtube.com/@GPUMODE
duration_seconds: 5002
level: advanced
prerequisites: [inference-engines, arithmetic-intensity-and-the-roofline-model]
takeaways:
  - Running several small GPU operations in sequence means writing each one's output to memory and reading it back in for the next, even though none of that intermediate data was ever actually needed anywhere else.
  - A fused kernel performs several operations in one pass while data stays in fast on-chip memory the whole time, eliminating the unnecessary round trips to slower main memory.
  - Fusion pays off more than most optimizations precisely because so many of a model's operations are small and memory-bound, which is exactly the regime where avoiding extra memory traffic matters most.
check_questions:
  - q: Why does running unfused operations waste time even if each individual operation is fast?
    a: Each operation's output has to be written to main memory and then read back in for the next operation, even when that intermediate result is never used for anything else. That round trip to memory is often slower than the actual computation, so the GPU spends more time moving data than computing with it.
  - q: Why does kernel fusion specifically help memory-bound operations the most?
    a: A memory-bound operation is already limited by how fast data can move, not by how much compute is available. Fusion directly reduces the amount of data movement required, which is precisely the resource a memory-bound operation is starved for — unlike a compute-bound operation, where memory movement was never the bottleneck to begin with.
deeper:
  - label: "Qooba — Implementing Fused SwiGLU FFN From Scratch in Triton"
    url: https://www.youtube.com/watch?v=zklv6OpoqWE
related_formulas: []
---

## The hidden cost of doing things one step at a time

A transformer's forward pass is built from many individual operations —
matrix multiplications, activation functions, normalisation steps — chained
one after another. Run naively, each of these operations reads its input
from GPU memory, computes its result, and writes that result back out to
memory before the next operation reads it back in again. For an intermediate
result that is never needed anywhere except as input to the very next step,
every one of those writes and reads is pure overhead: time spent moving data
that nothing outside this one chain of operations will ever use.

## What fusion actually does

**Kernel fusion** combines several of these operations into a single GPU
kernel that performs all of them in one pass, keeping intermediate results in
fast on-chip memory — close to the GPU's compute units — instead of round-
tripping them out to the GPU's much slower main memory and back. The
computation performed is identical to running the operations separately; what
changes is eliminating the unnecessary memory traffic between steps that
served no purpose except moving data from one operation to the next.

## Why the payoff is so large

This connects directly to
[arithmetic intensity and the roofline model](/roadmaps/inference-engineering/arithmetic-intensity-and-the-roofline-model).
Fusion does not reduce the amount of useful computation an operation
performs, but it does reduce the bytes of memory traffic required to perform
it, which directly raises arithmetic intensity. For an operation that was
memory-bound — sitting under the sloped memory roof, limited by data
movement rather than compute — that is exactly the constraint fusion
relieves. A compute-bound operation, by contrast, gains little from fusion,
since memory movement was never its bottleneck. So much of a transformer's
per-token decode work is memory-bound, as covered in
[memory-bound vs compute-bound](/roadmaps/inference-engineering/memory-bound-vs-compute-bound),
that fusion ranks among the most valuable optimizations available to a
serving engine. That is exactly why every major inference engine covered in
the [previous lesson](/roadmaps/inference-engineering/inference-engines)
implements extensive kernel fusion internally rather than leaving it to
chance.

## A concrete example of the chain being fused

A common target for fusion in a transformer is the sequence of operations
right after an attention or feed-forward block: adding the block's output
back to its input (a residual connection), normalising the result, and
applying an activation function, all one after another. Run unfused, that is
three separate memory round trips for what is conceptually one small step.
Fused into a single kernel, the GPU reads the input once, performs all three
operations while the data sits in fast on-chip memory, and writes the final
result out once — three memory round trips reduced to two, repeated for
every single one of these junctions throughout every layer of the network,
which is where the savings compound into something substantial across a
full forward pass.

## Not something you usually write yourself

In practice, most engineers using a mature inference engine never write a
fused kernel by hand — the engine's own optimized kernels already apply this
technique throughout the model's forward pass. Understanding what fusion is
and why it matters is valuable regardless, because it explains a real chunk
of why a mature engine outperforms a naive implementation so dramatically,
and it is directly relevant for anyone writing a custom operation — a new
activation function, a specialised attention variant — who wants that custom
code to perform anywhere near as well as the engine's own built-in kernels.
