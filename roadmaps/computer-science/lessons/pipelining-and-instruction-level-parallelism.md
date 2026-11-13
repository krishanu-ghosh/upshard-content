---
title: "Pipelining and Instruction-Level Parallelism"
description: How CPUs overlap the fetch, decode, execute, and writeback stages of different instructions, and why data, control, and structural hazards make this hard.
roadmap: computer-science
node: pipelining-and-instruction-level-parallelism
group: computer-architecture
video:
  provider: youtube
  id: "BVNx3wtJ9vs"
  start: 0
  end: null
creator:
  name: Computerphile
  channel_url: https://www.youtube.com/@Computerphile
duration_seconds: 1308
level: beginner
prerequisites:
  - memory-hierarchy-and-caching
takeaways:
  - "Pipelining overlaps the stages of different instructions rather than running one instruction fully to completion before starting the next, the same way a factory assembly line overlaps work on different units."
  - "A classic five-stage pipeline (fetch, decode, execute, memory access, writeback) can have up to five instructions in flight at once, each at a different stage, even though any single instruction still passes through all five stages in order."
  - "A data hazard happens when an instruction needs a result that a previous, still-in-flight instruction hasn't produced yet, which can force the pipeline to stall or require forwarding the result early."
  - "A control hazard happens because the CPU doesn't know which instruction to fetch next after a branch until that branch has been evaluated, which can be several stages later than when it was fetched."
  - "Branches are expensive specifically because of control hazards: if the CPU guesses the wrong direction (via branch prediction) or simply waits, it either has to flush several partially-completed instructions from the pipeline or stall it, both of which throw away work the pipeline already overlapped."
  - "A structural hazard happens when two instructions in the pipeline at the same time need the same piece of hardware simultaneously, such as both needing memory access in the same cycle, which forces one to wait even though neither is conceptually dependent on the other."
check_questions:
  - q: "If a non-pipelined CPU takes 5 cycles to fully execute one instruction, why doesn't a 5-stage pipelined CPU finish 5 instructions in 5 cycles total?"
    a: "The pipeline fills up gradually - the first instruction still takes 5 cycles to get through all 5 stages, since each stage has to happen in order for any single instruction. The speedup comes from overlap: by the time the first instruction reaches its second stage, a second instruction can enter the first stage, so in steady state the pipeline finishes roughly one instruction per cycle rather than one every 5 cycles - but there's still a fixed fill-up cost at the start."
  - q: "Why do branch instructions hurt pipeline performance more than, say, an addition instruction does?"
    a: "An addition's operands and result are known as soon as it's decoded and executed, so later instructions can usually proceed without waiting. A branch's outcome - which address to fetch next - often isn't known until it's evaluated, several stages after it was fetched, so the CPU has already fetched and started processing instructions that may turn out to be the wrong ones; if its guess is wrong, that partially-completed work has to be discarded and the pipeline has to restart from the correct target."
  - q: "Does adding more pipeline stages always make a CPU faster?"
    a: "Not unconditionally - more, shorter stages can increase clock speed since each stage does less work per cycle, but they also increase the penalty of hazards: a control hazard now means discarding more in-flight instructions when a branch is mispredicted, and a data hazard may need to stall for more cycles before a result is ready, so deeper pipelines trade a higher ceiling on throughput for a higher cost whenever that smooth overlap breaks down."
deeper:
  - label: "Advanced CPU Designs: Crash Course Computer Science #9"
    url: "https://www.youtube.com/watch?v=rtAlC5J1U40"
related_formulas: []
---

## Overlapping work instead of doing it in strict sequence

The fetch-decode-execute cycle from the first lesson in this group describes what happens to *one* instruction, run start to finish before the next one begins. A real CPU doesn't actually wait that long. Instead, it splits instruction processing into discrete stages - commonly **fetch**, **decode**, **execute**, **memory access**, and **writeback** - and runs different instructions through different stages *at the same time*, the way an assembly line has one car getting its engine installed while another, behind it, is having its doors fitted.

Picture five instructions, I1 through I5, flowing through a five-stage pipeline:

```
Cycle:   1    2    3    4    5    6    7    8    9
I1:      F    D    E    M    W
I2:           F    D    E    M    W
I3:                F    D    E    M    W
I4:                     F    D    E    M    W
I5:                          F    D    E    M    W
```

Each instruction still passes through all five stages, in order, taking five cycles start to finish - pipelining doesn't make any single instruction faster. What it does is let the hardware for each stage stay busy: while I1's result is being written back in cycle 5, I5 is only just being fetched. Once the pipeline is full, a new instruction can finish roughly every cycle instead of every five, which is where the throughput gain comes from. This is a form of **instruction-level parallelism** - extracting parallel work from what looks, in the source code, like a single sequential stream of instructions.

## When overlap breaks: hazards

Overlap only works cleanly when instructions are independent. When they aren't, you get a **hazard** - a situation where the next instruction can't safely proceed through the pipeline on schedule.

**Data hazards** occur when an instruction needs a value that an earlier, still-in-flight instruction hasn't produced yet. For example:

```
ADD  R1, R2, R3   ; R1 = R2 + R3
SUB  R4, R1, R5   ; R4 = R1 - R5   (needs R1 from the line above)
```

If `SUB` reaches its execute stage before `ADD` has written R1 back, it would read a stale value. CPUs handle this either by **forwarding** the result directly from the ALU's output to the stage that needs it (skipping the wait for writeback) or, when forwarding isn't possible, by **stalling** - inserting idle cycles ("bubbles") until the value is ready.

**Structural hazards** occur when two instructions in flight need the same physical hardware resource in the same cycle - for instance, if instruction fetch and a data memory access both need the same memory port at once. One has to wait, purely because the hardware is shared, regardless of whether the instructions have anything to do with each other logically.

**Control hazards** are about not knowing *what* to fetch next. Every pipeline fetches instructions ahead of knowing how earlier branches resolve. A conditional branch's outcome usually isn't known until it's been decoded and evaluated, stages after it was fetched - but the pipeline has already fetched something to follow it, guessing it would just continue sequentially.

## Why branches specifically are expensive

A branch forces the CPU to choose between two bad options. It can **stall** fetching until the branch resolves, which wastes cycles doing nothing even on a correctly-anticipated branch. Or it can **predict** which way the branch will go (modern CPUs use branch predictors that track history and guess with high accuracy) and speculatively fetch and start executing instructions down the predicted path.

Prediction pays off when it's right - no stall at all. But when the prediction is wrong, every instruction the pipeline had already pulled in down the wrong path has to be thrown away, and the pipeline has to restart from the correct target address. In a deep pipeline, several instructions can be in flight by the time a branch resolves, so a misprediction doesn't just cost one cycle - it costs however many cycles were spent on now-discarded work, called the **branch penalty**. This is exactly why loop-heavy, branch-heavy code can run noticeably slower than straight-line code doing a similar amount of arithmetic, and why branch prediction accuracy is one of the most closely optimized parts of real CPU design.
