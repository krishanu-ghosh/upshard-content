---
title: "Race Conditions and Thread Safety"
description: How unsynchronized access to shared state produces timing-dependent bugs, worked through a classic lost-update counter example.
roadmap: computer-science
node: race-conditions-and-thread-safety
group: concurrency-and-practices
video:
  provider: youtube
  id: "QAzuAn3nFGo"
  start: 0
  end: null
creator:
  name: Core Dumped
  channel_url: https://www.youtube.com/@CoreDumpped
duration_seconds: 660
level: intermediate
prerequisites:
  - concurrency-vs-parallelism
takeaways:
  - "A race condition happens when two or more threads read and modify shared state without coordination, and the final result depends on the unpredictable order in which their instructions interleave."
  - "A single line like `counter += 1` is not one atomic step — it's a read, a modify, and a write, and another thread can run between any two of those steps."
  - "The classic lost-update bug: two threads each read the same old value, each add one, and each write back the same new value, so one increment silently disappears."
  - "Race conditions are 'heisenbugs' — timing-dependent, so they can pass thousands of test runs and then appear in production under different scheduling or load."
  - "'Thread-safe' means a piece of code behaves correctly no matter how its operations interleave with other threads touching the same state, not merely that it hasn't failed yet in testing."
  - "The fix is always some form of making the critical section indivisible from other threads' point of view — a lock, an atomic instruction, or avoiding shared mutable state altogether."
check_questions:
  - q: "Why does incrementing a shared integer from two threads sometimes produce a correct result and sometimes lose an update, instead of failing consistently?"
    a: "Whether the bug appears depends on the exact timing of thread scheduling — if one thread's read-modify-write completes before the other starts, the result is correct, but if their steps interleave, both threads can read the same stale value and overwrite each other's work, and that timing varies run to run."
  - q: "If a function works correctly every time it's called from a single thread, why might it not be thread-safe?"
    a: "Single-threaded correctness only proves there's no issue when operations run one at a time in isolation; thread safety is about what happens when multiple threads execute the function's steps concurrently and interleave partway through, which single-threaded testing never exercises."
  - q: "Why doesn't just making an operation 'faster' (so the unsafe window is tiny) actually fix a race condition?"
    a: "A race condition is about correctness under any possible interleaving, not about probability — a tiny unsafe window can still be hit, especially at scale or under heavy load, so shrinking the window reduces how often the bug shows up without eliminating the bug itself."
deeper:
  - label: "Race Conditions (Real Python)"
    url: "https://realpython.com/python-race-conditions/"
related_formulas: []
---

## What a race condition actually is

A race condition occurs when the correctness of a program depends on the relative timing of events that aren't otherwise coordinated — specifically, when two or more threads access the same piece of shared, mutable state and at least one of them writes to it, with no mechanism forcing their operations to happen in a safe order. The word "race" is apt: the threads are racing to read and write the same memory, and whichever one happens to get there first (or interleave in a particular way) determines the outcome. Nothing about the program's *logic* is wrong in isolation — each thread, run alone, does exactly what it's supposed to. The bug only exists in the gap between threads.

## The classic example: losing an increment

Consider two threads that both execute `counter += 1` on a shared variable, starting from `counter = 0`, each running the increment 1,000 times. You'd expect the final value to be 2,000. It usually isn't — and the reason reveals the mechanism.

`counter += 1` looks like a single operation in source code, but at the hardware level it's at least three distinct steps:

1. **Read** the current value of `counter` from memory into a register.
2. **Modify** the value in the register (add 1).
3. **Write** the register's value back to `counter` in memory.

If Thread A and Thread B's increments never overlap, this works fine: A reads 41, writes 42; B reads 42, writes 43. But suppose the OS scheduler interrupts A right after step 1:

- Thread A reads `counter` (value is 41) into its own register.
- The scheduler switches to Thread B. B reads `counter` — still 41, because A hasn't written anything back yet — into its register, adds 1, and writes 42 back to memory.
- The scheduler switches back to Thread A, which still has 41 sitting in its register from before. It adds 1, getting 42, and writes 42 back to memory — overwriting B's write.

Two increments happened, but `counter` only advanced by one. B's update was silently lost. Run this with two threads doing 1,000 increments each, and you'll typically land somewhere below 2,000 — the exact number varies from run to run, because it depends on exactly how the scheduler happened to interleave the two threads' read-modify-write sequences that particular time. Run it again and you might get a different wrong answer, or even briefly get the right one by luck.

## Why these bugs are so hard to catch

This unpredictability is why race conditions are sometimes called "heisenbugs" — the act of observing them (running under a debugger, adding logging, slowing things down) often changes the timing enough to make the bug disappear, only for it to resurface later under production load with different thread counts, different CPU scheduling, or different data sizes. A race condition can sit dormant through months of testing and thousands of CI runs, because the specific interleaving that triggers it is rare — and then appear the moment real traffic creates enough contention for that interleaving to happen regularly. This is also why "it passed all the tests" is weak evidence that concurrent code is correct: passing tests only shows the interleavings that actually occurred during those runs behaved fine, not that every possible interleaving does.

## What "thread-safe" actually means

Calling code **thread-safe** means something precise: it behaves correctly regardless of how its operations are interleaved with other threads' operations on the same shared state, for every possible interleaving — not just the ones that happen to get exercised in testing. A function that only works when called from one thread at a time, or that happens to work under light load, is not thread-safe; it's just gotten lucky so far.

The fix for the counter example is to make the read-modify-write sequence indivisible from the perspective of other threads — a **critical section** that only one thread can be inside at a time. This is usually done with a **lock** (mutex): each thread must acquire the lock before touching `counter` and release it after, so a second thread attempting to increment has to wait until the first has finished its full read-modify-write cycle. An alternative is a hardware-provided **atomic operation** (like `fetch-and-add`), which performs the read, modify, and write as a single uninterruptible instruction so there's no window for another thread to interleave at all. A third option, often the best when it's available, is to avoid sharing mutable state in the first place — giving each thread its own counter and summing them at the end sidesteps the race entirely rather than managing it.

None of these fixes change *what* the program computes; they only remove the possibility of two threads' operations interleaving in a way that corrupts shared state — which is exactly what distinguishes "happens to work" from "actually thread-safe."
