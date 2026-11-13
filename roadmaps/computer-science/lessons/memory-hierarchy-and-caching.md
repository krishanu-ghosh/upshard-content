---
title: "The Memory Hierarchy and Why Caching Works"
description: Why computers stack registers, L1/L2/L3 cache, RAM, and disk into a hierarchy, and why locality is the reason caching actually works.
roadmap: computer-science
node: memory-hierarchy-and-caching
group: computer-architecture
video:
  provider: youtube
  id: "6JpLD3PUAZk"
  start: 0
  end: null
creator:
  name: Computerphile
  channel_url: https://www.youtube.com/@Computerphile
duration_seconds: 366
level: beginner
prerequisites:
  - how-a-cpu-executes-instructions
takeaways:
  - "A modern CPU can execute an instruction far faster than RAM can supply the data for it, so without something in between, the CPU would spend most of its time idle, waiting on memory."
  - "The memory hierarchy stacks several kinds of storage - registers, L1, L2, L3 cache, RAM, and disk - each level trading capacity and cost for speed, with the fastest, most expensive storage closest to the CPU and the slowest, cheapest storage furthest away."
  - "Caching only works because real programs exhibit locality: temporal locality means recently used data is likely to be reused soon, and spatial locality means data near recently used data is likely to be used soon too."
  - "A cache hit is answered directly from the cache at that level's speed; a cache miss has to fall through to the next, slower level, fetch the data, and typically store a copy in the cache on the way back for next time."
  - "Because caches work in fixed-size blocks rather than single bytes, spatial locality gets exploited automatically - loading one byte pulls in its whole neighborhood, which is why accessing memory in predictable, sequential patterns is so much faster than jumping around randomly."
check_questions:
  - q: "If RAM is already much faster than a disk, why isn't a single, larger RAM-speed cache the obvious solution instead of stacking multiple small cache levels?"
    a: "Speed and size trade off against each other at the hardware level - a cache fast enough to match the CPU's clock has to be physically tiny and sit extremely close to the CPU core, which caps how much of it you can build; stacking several progressively larger, progressively slower levels (L1, L2, L3) lets most accesses still get served quickly while still backing up a reasonably large working set, which one huge fast cache can't do."
  - q: "A program that processes a huge array strictly from start to end tends to run faster than one that jumps to random indices in the same array. Why does the memory hierarchy explain this?"
    a: "Sequential access has strong spatial locality: once the first element of a cache block is loaded, the CPU gets the whole block, including nearby elements, as a side effect, so later accesses to those nearby elements are cache hits. Random access rarely reuses a block it just loaded, so it causes far more cache misses and far more slow trips down to RAM."
  - q: "What actually happens, mechanically, on a cache miss?"
    a: "The requested address isn't found in that cache level, so the request falls through to the next, slower level of the hierarchy (the next cache level, or RAM); once the data is retrieved, it's typically written into the cache that missed, so a repeat access to the same or nearby data becomes a hit instead of another slow fetch."
deeper:
  - label: "Locality of Reference - GeeksforGeeks"
    url: "https://www.geeksforgeeks.org/computer-organization-architecture/locality-of-reference-and-cache-operation-in-cache-memory/"
related_formulas: []
---

## The problem the hierarchy exists to solve

A CPU core running at a few gigahertz can execute an instruction every fraction of a nanosecond. A round trip to RAM takes on the order of 100 nanoseconds. If every instruction had to wait for a full RAM access, the CPU would spend the overwhelming majority of its time sitting idle - all that clock speed would be wasted waiting on memory rather than computing.

The fix isn't to make RAM faster (physics and cost make that hard) but to put something faster, smaller, and far more expensive per byte between the CPU and RAM, and to only go all the way out to RAM when that faster storage doesn't already have what's needed. That's the **memory hierarchy**: registers at the top, then L1 cache, then L2 cache, then often a shared L3 cache, then RAM, then disk at the bottom - each level larger, cheaper per byte, and slower than the one above it.

- **Registers** hold a handful of values, read or written in a single clock cycle, built directly into the CPU core.
- **L1 cache** is tiny (often 32-64 KB per core) but nearly as fast as registers, private to each core.
- **L2 cache** is larger (several hundred KB to a few MB) and a bit slower, often still private to each core.
- **L3 cache** is larger still (tens of MB) and shared across all the cores on a chip, trading more speed for more capacity.
- **RAM** holds gigabytes, is far cheaper per byte than any cache, but costs roughly 100 nanoseconds per access.
- **Disk** (SSD or spinning platter) holds terabytes for very little money per byte, but an access can cost anywhere from tens of microseconds to several milliseconds - thousands of times slower than RAM.

Each level exists because no single technology gives you speed, size, and low cost all at once. Fast memory is physically expensive to build at scale, so the hierarchy approximates "lots of memory at register speed" by keeping only the data that's actually in use at the fast end, and pushing everything else down to cheaper, slower storage.

## Why caching works at all: locality

Stacking cache levels only pays off if a small, fast cache can usually answer the CPU's requests without reaching further down the hierarchy. That depends entirely on how real programs actually access memory, which is governed by two patterns:

**Temporal locality** - if a program reads a variable now, it's likely to read that same variable again soon. Loop counters, frequently-called functions, and repeatedly-checked flags are all temporal locality in action. A cache that keeps recently-used data around is betting on exactly this pattern.

**Spatial locality** - if a program reads one memory address, it's likely to read a nearby address soon too. Arrays processed from start to end, struct fields accessed together, and instructions executed in sequence from memory all exhibit this. Caches exploit spatial locality structurally: they don't fetch single bytes, they fetch fixed-size **blocks** (commonly 64 bytes) at a time, so one access pulls in an entire neighborhood of data on the assumption that neighbors will be wanted next.

If programs accessed memory in a genuinely random pattern, none of this would help - every access would be a coin flip on whether the needed data happened to already be cached, and caching would just add overhead. Real programs are nowhere near random, because loops revisit the same instructions and variables, and because data structures like arrays and structs are laid out so that related fields sit physically close together. Locality is an empirical fact about how software behaves, not a law of physics - but it holds overwhelmingly often enough that the entire memory hierarchy is built around betting on it.

## Hits, misses, and what a miss actually costs

A **cache hit** means the requested address is already present in a given cache level; the CPU gets its data at that level's speed and never bothers the slower levels below it. A **cache miss** means it isn't there, so the request falls through to the next level down - L1 miss checks L2, L2 miss checks L3, L3 miss finally goes to RAM. When the data is eventually found, it's usually copied up into the caches that missed, so that this same address (and its neighboring block) will be a hit the next time around.

The cost of a miss compounds with depth: an L1 miss that hits in L2 might cost ten times longer than an L1 hit; a miss that falls all the way to RAM can cost over a hundred times longer. This is why "cache-friendly" code - processing data sequentially, reusing values instead of recomputing them, keeping working sets small - can run many times faster than code doing the same logical work in a way that defeats locality, even though both versions execute roughly the same number of instructions.
