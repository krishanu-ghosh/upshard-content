---
title: "Arrays, Linked Lists, Stacks, and Queues"
description: The four linear data structures every other structure builds on, and the time-complexity tradeoffs that decide which one to reach for.
roadmap: datastructures-and-algorithms
node: arrays-linked-lists-stacks-and-queues
group: core-data-structures
video:
  provider: youtube
  id: "njTh_OwMljA"
  start: 0
  end: null
creator:
  name: HackerRank
  channel_url: https://www.youtube.com/@HackerrankOfficial
duration_seconds: 420
level: beginner
prerequisites:
  - asymptotic-notation-big-o-theta-and-omega
takeaways:
  - "An array stores elements in one contiguous block of memory, which is what makes indexed access O(1) — the address of element i is just the base address plus i times the element size."
  - "A linked list trades that O(1) access for O(1) insertion and deletion at a known position, because elements live wherever memory is free and are connected by pointers instead of position."
  - "A stack is last-in-first-out: push and pop both happen at the same end, which is why it's the structure behind function call frames and \"undo\" history."
  - "A queue is first-in-first-out: items are added at the back and removed from the front, matching anything processed in arrival order, like a print queue or a task scheduler."
  - "None of these four structures is strictly better than the others — the right choice depends on whether your workload reads by position, grows and shrinks at the ends, or processes in a fixed order."
check_questions:
  - q: Why is reading `array[500000]` just as fast as reading `array[0]`, but finding the 500,000th node of a linked list is not?
    a: An array's elements sit in one contiguous block, so the address of any index is computed directly from the base address with simple arithmetic — no traversal needed. A linked list has no such formula; the only way to reach the 500,000th node is to follow 500,000 pointers one at a time from the head.
  - q: You need to undo the last action a user took, and only the last one at a time. Which structure fits, and why?
    a: A stack. Undo is inherently last-in-first-out — the most recent action is always the first one you reverse — which is exactly the access pattern a stack's push/pop at one end provides.
  - q: Inserting into the middle of an array of a million elements is slow. Inserting into the middle of a linked list, given a pointer to the right spot, is fast. Why does the array pay a cost the list doesn't?
    a: Because an array's elements must stay contiguous, inserting in the middle means shifting every element after the insertion point over by one slot — an O(n) operation. A linked list just rewires two pointers at the insertion point, an O(1) operation once you're already there.
related_formulas: []
---


## Four structures, one axis of tradeoffs

Arrays, linked lists, stacks, and queues are usually taught as four separate things, but they're better understood as four points on a single axis: how much do you pay to read by position, versus how much do you pay to insert or remove?

An **array** is a block of memory reserved all at once, with elements packed one after another. Because the block is contiguous, the computer can calculate the memory address of any element directly — `base_address + index * element_size` — without looking at anything else. That's what makes `array[i]` an O(1) operation regardless of how large the array is or which index you ask for. The cost shows up on the other side: inserting a new element in the middle means every element after it has to physically shift over by one slot to make room, which is O(n) in the worst case. Appending to the end is usually O(1), unless the array's underlying block is full and has to be reallocated to a bigger one elsewhere in memory — a cost that dynamic arrays (like Python's list or Java's ArrayList) amortize across many appends so it averages out to O(1) per operation even though any single append might trigger a resize.

A **linked list** flips that tradeoff. Instead of one contiguous block, each element ("node") is a separate allocation holding its data plus a pointer to the next node (and, in a doubly linked list, a pointer to the previous one too). Nothing requires the nodes to be near each other in memory — they can be scattered anywhere, connected only by pointers. That removes the need to shift anything: inserting or deleting a node, given a pointer to the right spot, is O(1) — you just rewire two pointers. The cost is that there's no formula for "the address of element i" anymore. Reaching the nth node means starting at the head and following n pointers, one hop at a time, which is O(n). A linked list is fast to edit once you're there and slow to get there; an array is fast to get anywhere and slow to edit in the middle.

## Stacks: last in, first out

A **stack** restricts access to one end of a sequence: you can only `push` (add) or `pop` (remove) from the top. That restriction is the whole point — it guarantees last-in-first-out (LIFO) ordering, which matches a surprising number of real problems. When a function calls another function, the computer pushes a new call frame onto the **call stack**; when that function returns, its frame pops off and control resumes exactly where it left off in the caller. Undo history, the back button in a browser, and balanced-parentheses checking all follow the same LIFO shape. A stack can be implemented on top of either an array (with push/pop at the end, both O(1) amortized) or a linked list (push/pop at the head, both O(1)) — the restriction is a behavioral contract, not a specific underlying structure.

## Queues: first in, first out

A **queue** also restricts access, but to opposite ends: items are added at the back (`enqueue`) and removed from the front (`dequeue`), giving first-in-first-out (FIFO) ordering. Anything that should be processed in the order it arrived is a queue: a print spooler, a task scheduler, a breadth-first search frontier, requests waiting on a web server. Implementing a queue efficiently on an array takes a little more care than a stack does — removing from the front of a plain array would mean shifting everything else down, which is O(n) — so array-backed queues typically use a circular buffer that wraps the front and back pointers around the array instead of shifting data, keeping both enqueue and dequeue O(1). A doubly linked list sidesteps the issue entirely, since adding or removing from either end is O(1) without any shifting.

## Picking one

The decision almost always comes down to the same two questions: how will this data be accessed, and at which ends does it grow or shrink? If you need to look up elements by position frequently and changes mostly happen at the end, an array (or dynamic array) wins. If you're frequently inserting and removing from the middle or front, or the final size is unpredictable and you want to avoid resizing costs, a linked list is the better fit. If access is restricted by design — only ever the most recent item, or only ever the oldest — a stack or queue names that restriction explicitly, which also makes the code's intent clearer to the next person who reads it. These four structures reappear as building blocks throughout the rest of this roadmap: hash tables use arrays and linked lists together to resolve collisions, and graph traversal algorithms use a stack (for depth-first search) or a queue (for breadth-first search) to decide what to visit next.
