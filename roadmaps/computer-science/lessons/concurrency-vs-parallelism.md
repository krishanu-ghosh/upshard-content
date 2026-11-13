---
title: "Concurrency vs. Parallelism"
description: Why structuring a program as independent tasks (concurrency) is a different idea from running tasks at the same instant (parallelism).
roadmap: computer-science
node: concurrency-vs-parallelism
group: concurrency-and-practices
video:
  provider: youtube
  id: "RlM9AfWf1WU"
  start: 0
  end: null
creator:
  name: ByteByteGo
  channel_url: https://www.youtube.com/@ByteByteGo
duration_seconds: 253
level: intermediate
prerequisites:
  - synchronization-locks-and-semaphores
takeaways:
  - "Concurrency is about structure: a program is organized as multiple logically-independent tasks that can start, run, and finish in overlapping time windows."
  - "Parallelism is about execution: multiple tasks genuinely run at the same physical instant, which requires multiple cores or processors."
  - "A single-core CPU can be concurrent but never truly parallel — it creates the illusion of simultaneity by rapidly time-slicing between tasks."
  - "A single task can be parallel without being concurrent in any interesting sense — SIMD instructions apply one operation to many data elements at once with no independent tasks to coordinate."
  - "Concurrent code is what makes parallel execution possible on multi-core hardware, but concurrency itself doesn't require more than one core to exist."
  - "Confusing the two leads to wasted effort — adding threads to code that's concurrent but has no real parallel work to do won't speed anything up, and can even slow it down through coordination overhead."
check_questions:
  - q: "A video player decodes audio and video on a single-core embedded chip, switching between the two tasks thousands of times per second. Is this concurrent, parallel, both, or neither?"
    a: "It's concurrent but not parallel — the tasks are structured as independent units and appear to progress together, but on a single core only one instruction executes at any given instant, so there's no true simultaneity."
  - q: "Why can a 4-core CPU run four concurrent tasks in parallel, but not forty?"
    a: "Parallelism is bounded by the number of execution units actually available — four cores can genuinely execute four instruction streams at the same instant, but a fifth or fortieth task has to wait or be time-sliced in in, which turns the rest back into interleaved concurrency rather than parallelism."
  - q: "If a program has no concurrency in its design at all — just one long sequential loop — can it still benefit from parallel hardware?"
    a: "Yes, through data parallelism: a single operation like adding two arrays element-by-element can be split across SIMD lanes or multiple cores without restructuring the program into independent logical tasks, which is exactly why parallelism doesn't require concurrency."
deeper:
  - label: "Concurrency is not Parallelism (Rob Pike, Go Blog writeup)"
    url: "https://go.dev/blog/waza-talk"
related_formulas: []
---

## Two different questions

"Concurrency" and "parallelism" get used interchangeably so often that it's worth stating the distinction as plainly as possible: **concurrency is a property of a program's structure**, and **parallelism is a property of its execution**. Concurrency asks "can this program be broken into pieces that don't have to run in a fixed order relative to each other?" Parallelism asks "are multiple pieces of work physically happening at the same instant in time?" A program can be concurrent without ever being parallel, and a computation can be parallel without having any concurrent structure at all. They're independent axes, not two names for the same thing.

## Concurrency without parallelism: one core, many tasks

Take a single-core CPU running an operating system with multiple processes — say, a web browser and a music player. There is exactly one execution unit. At any given nanosecond, it is running instructions from the browser or from the music player, never both. And yet the system feels like both are running "at once": audio keeps playing while a page keeps loading.

This works because the OS **time-slices**: the scheduler gives the browser a few milliseconds of CPU time, then performs a context switch — saving the browser's registers and program counter, loading the music player's — and gives the music player its few milliseconds, then switches back. Each slice is far too fast for a human to perceive, so the two tasks appear simultaneous even though, mechanically, only one instruction stream is ever actually executing. This is concurrency in its purest form: the program (or system) is structured as independent tasks that can be interleaved in any order, but nothing requires more than one thing to physically happen at once. A single-core machine can run an arbitrarily concurrent program — it just can't run any part of it in parallel.

## Parallelism without much concurrency: SIMD and data parallelism

Now flip it around. Suppose you need to add two arrays of one million floating-point numbers, element by element. There's no interesting independent-task structure here — it's the same operation, repeated. But a modern CPU doesn't have to perform that addition one pair at a time. **SIMD** (Single Instruction, Multiple Data) hardware can load, say, 8 floats from each array into a wide register and execute one `add` instruction that produces 8 results simultaneously, in the same clock cycles it would otherwise take to do one. GPUs push this further, executing the same instruction across thousands of data-parallel lanes at once.

This is parallelism — real, simultaneous execution — achieved without decomposing the problem into independent logical tasks that need scheduling, synchronization, or communication. There's one instruction stream; it just operates on many pieces of data at the same instant. Compilers can often generate this kind of **data parallelism** automatically (auto-vectorization) from a plain sequential loop, which is precisely why you don't need to design "concurrent" code — multiple coordinating tasks — to benefit from it.

## Why the distinction matters in practice

The reason this isn't pedantry is that the two problems have different solutions. If your program spends its time waiting — on disk I/O, network responses, user input — restructuring it as concurrent tasks (threads, async callbacks, coroutines) lets the CPU do other useful work during the wait, even on a single core. Adding more cores doesn't fix a waiting problem; better concurrency structure does.

If instead your program is CPU-bound and doing the same kind of computation over and over — image processing, matrix multiplication, physics simulation — the win comes from parallelism: more cores, more SIMD lanes, more GPU threads genuinely executing simultaneously. Throwing concurrent structure (more threads) at a CPU-bound task running on one core doesn't make it faster; the core still executes one instruction at a time, and now you're paying context-switch overhead on top.

Rob Pike's famous framing is the cleanest summary: concurrency is about dealing with lots of things at once; parallelism is about doing lots of things at once. A well-designed concurrent program — one broken into independent, loosely-coupled tasks — is *easier* to run in parallel when multiple cores are available, because the hard work of figuring out what can safely happen out of order has already been done. But concurrency doesn't require parallel hardware to be useful, and parallel hardware doesn't require concurrent design to be exploited.

Once you can tell which problem you actually have, the next lesson follows directly. Race conditions only arise once genuinely concurrent (or parallel) tasks start touching the same shared state.
