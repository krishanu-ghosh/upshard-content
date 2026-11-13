---
title: "LRU Cache Design and Implementation"
description: Implement an LRU cache with O(1) get and put by pairing a hash map for lookup with a doubly linked list for eviction order.
roadmap: datastructures-and-algorithms
node: lru-cache-design-and-implementation
group: more-advanced-structures
video:
  provider: youtube
  id: "7ABFKPK2hD4"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 1069
level: intermediate
prerequisites: ["arrays-linked-lists-stacks-and-queues", "hash-tables-and-tries"]
takeaways:
  - "An LRU cache needs O(1) get and O(1) put while always being able to evict the least-recently-used entry, which neither a plain hash map nor a plain linked list can do alone."
  - "The hash map stores key -> node pointer, giving O(1) lookup of any cached key's position; the doubly linked list stores actual recency order, with the front as most-recently-used and the back as least-recently-used."
  - "A doubly linked list lets a node be unlinked from wherever it sits and relinked at the front in O(1), using only that node's prev/next pointers — no traversal needed, which is why it has to be doubly (not singly) linked."
  - "Every get that hits moves the accessed node to the front of the list, since being read counts as 'used'; every put either updates-and-moves an existing node or inserts a new one at the front, evicting the back node if capacity is exceeded."
  - "Using sentinel head and tail dummy nodes removes the need for special-case null checks when inserting at the front or evicting from the back, simplifying the pointer rewiring."
check_questions:
  - q: "Why does an LRU cache need both a hash map and a linked list, rather than just one or the other?"
    a: "A hash map alone gives O(1) get/put but has no notion of access order, so finding the least-recently-used entry to evict would require tracking order some other way, which the map itself can't do. A linked list alone can maintain recency order with O(1) move-to-front and evict-from-back, but finding where a given key sits in the list to move it would require an O(n) scan. Combining them gives O(1) for both: the hash map finds the node instantly, the list tracks and updates recency instantly."
  - q: "Why must the linked list be doubly linked instead of singly linked?"
    a: "Moving an accessed node to the front requires unlinking it from its current position, which means connecting its previous neighbor directly to its next neighbor. With a singly linked list you only have a pointer forward from a node, not backward to whatever points at it, so finding its predecessor to patch over the gap would require scanning from the head — O(n). A doubly linked list stores both prev and next pointers on every node, so removing it from the middle and splicing it elsewhere is O(1), with no scan needed."
  - q: "Why does accessing a key with get() have to modify the list even though it doesn't change the cached value?"
    a: "LRU eviction policy is defined entirely by recency of use, and a get() counts as a use just as much as a put() does. If get() didn't move the accessed node to the front, a frequently-read key that's never re-written would gradually drift toward the back of the list and eventually get evicted as if it were unused — which defeats the entire point of 'least recently used.' Moving it to the front on every successful get keeps the list's order a true reflection of actual access recency."
deeper:
  - label: "cp-algorithms style reference: LeetCode 146, LRU Cache"
    url: "https://leetcode.com/problems/lru-cache/"
related_formulas: []
---

## The problem the combination solves

An LRU (least-recently-used) cache holds a fixed number of key-value pairs. `get(key)` returns the value if present (and counts as "using" that key); `put(key, value)` inserts or updates a value, and if the cache is already at capacity, it must evict whichever entry was used least recently before inserting the new one. Both operations need to run in O(1), which immediately rules out any design that scans the whole cache to find the right entry or the right thing to evict.

Neither a hash map nor a linked list alone gets there. A hash map gives O(1) access by key but has no concept of "which key was used most recently" — that information would have to live somewhere else, and updating it on every access would cost as much as maintaining a separate ordered structure badly. A linked list can maintain a clean ordering — most-recently-used at the front, least-recently-used at the back — and moving a node to the front or dropping the back node is O(1) if you already have a pointer to the node. But finding the node for a given key inside the list, without the map, means walking from one end — O(n).

Combine them and each covers the other's gap: the hash map stores `key -> pointer to that key's node in the linked list`, so any key's node is found in O(1); the doubly linked list stores actual recency order, so once you have the node, moving or removing it is O(1) too.

## Why doubly linked, specifically

Removing a node from the middle of a linked list to move it to the front requires reconnecting its two neighbors to each other, skipping over the removed node. That means you need to know both what comes before the node and what comes after it. A singly linked list only stores a forward pointer, so finding "what points at this node" requires scanning from the head — exactly the O(n) cost we're trying to avoid. A doubly linked list stores both `prev` and `next` on every node, so unlinking a node is a handful of pointer reassignments using information the node already carries, independent of where in the list it sits.

Implementations typically add two sentinel (dummy) nodes — a fixed `head` and `tail` that never hold real data — so the "most recently used" slot is always `head.next` and the "least recently used" slot is always `tail.prev`. This removes special-casing for an empty list or for inserting/removing at an actual boundary: every real node always has a genuine `prev` and `next` to work with.

## Walking through a sequence of operations

Say capacity is 2. Operations in order: `put(1, "a")`, `put(2, "b")`, `get(1)`, `put(3, "c")`, `get(2)`.

- `put(1, "a")`: cache is empty, under capacity. Insert node `(1,"a")` at the front. List (front to back): `(1,"a")`. Map: `{1 -> node1}`.
- `put(2, "b")`: under capacity. Insert `(2,"b")` at the front. List: `(2,"b"), (1,"a")`. Map: `{1 -> node1, 2 -> node2}`.
- `get(1)`: key 1 is present — this is a use, so move `node1` to the front. List: `(1,"a"), (2,"b")`. Return `"a"`.
- `put(3, "c")`: cache is at capacity (2 entries) and key 3 is new, so evict the back of the list — that's `(2,"b")` — removing it from both the list and the map. Then insert `(3,"c")` at the front. List: `(3,"c"), (1,"a")`. Map: `{1 -> node1, 3 -> node3}`. Key 2 is gone.
- `get(2)`: key 2 is no longer in the map — miss. Return not-found.

Every step touched only the front or back of the list, or a node the map pointed to directly — never a scan. That's the whole mechanism: the map answers "where," the list answers "in what order," and together they keep both get and put at O(1), the design every interview-style LRU implementation converges on.
