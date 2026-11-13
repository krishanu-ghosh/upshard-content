---
title: "Deques and Circular Buffers"
description: How a double-ended queue is implemented as a ring buffer over a fixed array, and why wraparound indices make that fast.
roadmap: datastructures-and-algorithms
node: deques-and-circular-buffers
group: advanced-linear-structures
video:
  provider: youtube
  id: "hOPYEI_d50s"
  start: 0
  end: null
creator:
  name: CS Hero
  channel_url: https://www.youtube.com/@cshero365
duration_seconds: 660
level: intermediate
prerequisites: []
takeaways:
  - "A deque (double-ended queue) supports O(1) push and pop at both the front and the back, which is strictly more than a stack (one end) or a plain queue (push one end, pop the other) each support alone."
  - "A circular buffer implements a deque over a single fixed-size array by letting the front and back indices wrap around to 0 after reaching the array's end, using modulo arithmetic instead of shifting elements."
  - "Wraparound avoids the O(n) cost of shifting every remaining element forward that a naive array-based queue pays every time it dequeues from index 0."
  - "A circular buffer must track size (or reserve one empty slot) separately from its front/back indices, because front == back is ambiguous between 'completely empty' and 'completely full' once wraparound is in play."
  - "Real systems lean on this structure constantly: BFS uses a deque to get O(1) pops from the front instead of a list's O(n) pop-from-front; sliding-window algorithms use it to evict from both ends; and task schedulers use fixed-capacity ring buffers so producers and consumers can operate without reallocating memory under load."
check_questions:
  - q: "Why can't a plain array-based queue just use index 0 as the front and shift everything left after every dequeue?"
    a: "Shifting every remaining element one slot to the left after each dequeue is O(n) work per dequeue, since every element after the removed one has to move. That turns a sequence of n dequeues into O(n²) total work. A circular buffer avoids this entirely by never moving existing elements — it just advances the front index (wrapping around to 0 if it runs off the end of the array), making dequeue O(1) regardless of how many elements are left."
  - q: "In a circular buffer, why is front == back not enough information to tell whether the buffer is empty or full?"
    a: "Both states can produce front and back pointing at the same slot: a brand-new empty buffer starts that way, but a full buffer that has wrapped all the way around also ends up with back having circled back to meet front. Without extra information, the two cases are indistinguishable from the indices alone. Implementations resolve this either by tracking a separate size/count field alongside the two indices, or by deliberately always leaving one slot unused so a full buffer never lets back catch up to front."
  - q: "Why does BFS specifically benefit from a deque rather than, say, a plain array-based list?"
    a: "BFS needs to repeatedly remove from the front of its frontier (the node discovered earliest) while adding new neighbors to the back, every single iteration. An array-backed list's pop-from-front is O(n) because it has to shift every remaining element, which would turn a BFS over n nodes into O(n²) just from queue overhead. A deque, implemented as a circular buffer or doubly linked list, makes both the front-removal and back-insertion O(1), so the queue operations don't become the bottleneck of the traversal."
deeper:
  - label: "Data Structures: How to implement a Circular Array (ring buffer)? (CS Hero)"
    url: "https://www.youtube.com/watch?v=dgn_KBx68Fc"
related_formulas: []
---

## What a deque adds over a stack or a queue

A **deque** (pronounced "deck," short for double-ended queue) supports four O(1) operations: push to the front, push to the back, pop from the front, and pop from the back. A stack only gives you one end; a plain queue gives you push on one end and pop on the other, but not push-front or pop-back. A deque is the superset — anywhere a stack or a queue is needed, a deque can substitute, which is why many standard libraries (Python's `collections.deque`, C++'s `std::deque`, Java's `ArrayDeque`) implement deque as the general-purpose default and let stack/queue usage just ignore the operations they don't need.

The implementation challenge is doing all four operations in O(1) using a plain array as backing storage, since a naive array only gives you O(1) at one end (the back) for free — inserting or removing at the front normally means shifting every other element over.

## The circular buffer: wraparound instead of shifting

A **circular buffer** (also called a ring buffer) solves this by treating a fixed-size array as if its last index were adjacent to its first — conceptually a ring, not a line. Two indices, typically called `front` and `back`, track the current ends of the live data. Instead of shifting elements, every push or pop just moves one of these indices, wrapping around to 0 using the modulo operator whenever it would otherwise run past the array's final index.

Concretely, with an array of capacity `C`:

- **Push back:** write the new element at `back`, then set `back = (back + 1) % C`.
- **Pop front:** read the element at `front`, then set `front = (front + 1) % C`.
- **Push front:** set `front = (front - 1 + C) % C`, then write the new element there (the `+ C` before the modulo handles the case where `front` was 0 and would otherwise go negative).
- **Pop back:** set `back = (back - 1 + C) % C`, then read the element there.

Every one of these is a constant number of arithmetic operations and one array access — no element ever moves once written, and nothing is shifted. Compare this to a naive queue that always pops from index 0: after removing one element, every remaining element has to shift left by one slot to keep index 0 as the front, which is O(n) per pop. Wraparound sidesteps that completely by letting the "front" conceptually travel around the array instead of forcing the data to travel to meet a fixed front.

## The empty-vs-full ambiguity

There's a subtlety that trips up a lot of first implementations: once `front` and `back` can both wrap around, the condition `front == back` becomes ambiguous. A freshly initialized, completely empty buffer has `front == back` (both pointing at index 0, say). But a buffer that has been filled to capacity and had its `back` index wrap all the way back around to meet `front` *also* ends up with `front == back`. The indices alone can't distinguish "nothing here" from "completely full."

Two standard fixes exist. The simplest is to maintain a separate `size` counter alongside `front` and `back`, incrementing it on every push and decrementing it on every pop — now emptiness and fullness are just `size == 0` and `size == C`, no ambiguity. The alternative, used when avoiding an extra field matters, is to deliberately treat the buffer as full when it has `C - 1` elements rather than `C`, permanently sacrificing one slot so that `back` can never advance far enough to equal `front` while genuinely full — full and empty remain distinguishable by the gap between the indices alone, at the cost of one unusable slot.

## Where this shows up in practice

**Breadth-first search** repeatedly pops the earliest-discovered node from the front of its frontier while pushing newly discovered neighbors to the back, every iteration, for the entire traversal. If that frontier were a plain array-backed list, every pop-from-front would be O(n), turning the whole traversal's queue overhead into O(n²). A deque backed by a circular buffer (or a doubly linked list) keeps both operations O(1), so the queue never becomes the bottleneck.

**Sliding-window algorithms**, like the monotonic-deque approach to sliding window maximum, need to evict elements from both ends — the back when a smaller element is superseded, the front when an element ages out of the window — which is precisely the two-ended eviction a deque is built for.

**Task schedulers and producer-consumer pipelines** frequently use a fixed-capacity ring buffer as a work queue: producers push tasks to the back, workers pop from the front, and because the buffer's size is fixed up front, the system never pays for a dynamic array's occasional resize — at the cost of needing a policy (block, drop, or overwrite) for what happens when the ring fills up.
