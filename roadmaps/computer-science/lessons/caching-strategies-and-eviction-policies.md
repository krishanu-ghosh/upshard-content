---
title: "Cache Eviction: LRU, LFU & MFU"
description: Which cached item to throw out when the cache is full — least recently used, least frequently used, or most frequently used — plus Huffman coding.
roadmap: computer-science
node: caching-strategies-and-eviction-policies
group: representation-and-caching
video:
  provider: youtube
  id: "bLEIHn-DgoA"
  start: 0
  end: null
creator:
  name: NeetCodeIO
  channel_url: https://www.youtube.com/@NeetCodeIO
duration_seconds: 885
level: intermediate
prerequisites: []
takeaways:
  - A cache with a fixed capacity needs an eviction policy to decide what to remove once it's full and a new item needs to be added.
  - LRU (least recently used) evicts whatever hasn't been accessed in the longest time, on the theory that recent access predicts near-future access, and it's implementable in O(1) per operation with a hash map plus a doubly linked list.
  - LFU (least frequently used) evicts whatever has been accessed the fewest total times, which protects items with a long history of heavy use even if they haven't been touched in the last few seconds.
  - MFU (most frequently used) evicts the opposite of LFU — the most-accessed item — which sounds backwards but fits workloads where an item's usefulness drops off sharply right after a burst of access.
  - Huffman coding builds an optimal prefix-free binary code for a set of symbols by repeatedly merging the two least-frequent symbols into a tree, giving common symbols short codes and rare symbols long ones without any code being a prefix of another.
check_questions:
  - q: Why does LRU need both a hash map and a doubly linked list to achieve O(1) operations, rather than just one or the other?
    a: The hash map gives O(1) lookup of any item's node by key, and the doubly linked list gives O(1) removal and re-insertion of a node at the "most recently used" end, which together let both "get" and "put" update recency in constant time — a hash map alone can't track order, and a linked list alone can't look up a key in O(1).
  - q: In what situation would LFU keep an item in cache that LRU would have already evicted?
    a: An item that was accessed thousands of times yesterday but hasn't been touched in the last hour would be high-priority under LFU (high total frequency) but low-priority under LRU (long time since last access), so LFU keeps it while LRU would evict it.
  - q: Why can't any Huffman code be a prefix of another code in the same encoding?
    a: If one code were a prefix of another, a decoder reading bits one at a time couldn't tell whether it had finished reading the shorter code or was partway through the longer one, so the prefix-free property is what makes the encoded bitstream unambiguous to decode.
deeper:
  - label: "LRU Cache"
    url: "https://www.youtube.com/watch?v=VLdtHsCX2yA"
  - label: "How Huffman Trees Work - Computerphile"
    url: "https://www.youtube.com/watch?v=umTbivyJoiI"
related_formulas: []
---

## Every cache eventually has to say no

A cache speeds things up by keeping a small, fast copy of frequently-needed data close at hand instead of re-fetching or recomputing it every time. But "small" means finite, and a finite cache fills up. Once it's full and a new item needs to go in, something already there has to leave — the choice of *what* to evict is the eviction policy, and different policies suit different access patterns.

## LRU: recency as the signal

Least Recently Used assumes that whatever hasn't been touched in the longest time is the least likely to be needed again soon. That's a reasonable bet for most everyday workloads, like a browser cache or a database buffer pool, where recently-viewed pages tend to get re-viewed. The elegant part is the implementation: a hash map from key to a node in a doubly linked list, where the list is kept ordered from most-recently-used (one end) to least-recently-used (the other). On every access, the map finds the node in O(1), and the list lets that node be unlinked and relinked at the "most recent" end in O(1) as well, with no scanning required. When the cache is full, the node at the "least recent" end is the one evicted, also O(1). This hash-map-plus-linked-list combination is the canonical answer to "design an LRU cache," precisely because neither structure alone gets you both fast lookup and fast reordering.

## LFU: total usage as the signal

Least Frequently Used instead tracks how many times each item has ever been accessed, and evicts whichever has the lowest count. This protects items with a long track record of heavy use even during a lull — a reference dataset that's queried constantly in bursts, say, shouldn't get evicted just because nobody's queried it in the last ten seconds. The tradeoff is that LFU can get "stuck" favoring an old item that was popular once but is now irrelevant, since its historical count stays high even as its actual future usefulness has dropped to zero. Some LFU implementations add aging — periodically decaying counts — specifically to counter this.

## MFU: frequency, inverted

Most Frequently Used evicts the item with the *highest* access count — the opposite instinct from LFU. It sounds backwards until you picture a workload where items are read heavily right after being created and then essentially never again, like a streaming log buffer: an item that's already racked up a lot of accesses has likely already delivered its value and is unlikely to be read much more, while a newer, lower-count item is more likely to still be in its active phase. MFU is far less common than LRU or LFU, but it's a useful reminder that "frequency" isn't automatically the same signal as "future usefulness" — the right policy depends on how a specific workload's access pattern actually behaves over time.

## Huffman coding: a different kind of "most used wins"

Huffman coding isn't an eviction policy, but it shares the same underlying idea as LFU in reverse: give the things that are used most *the cheapest treatment*. Given a set of symbols and how often each occurs, Huffman coding builds a binary tree bottom-up — repeatedly taking the two least-frequent symbols (or partial trees) and merging them under a new parent node, until only one tree remains. Walking from the root to a symbol's leaf, recording a 0 for every left branch and a 1 for every right branch, gives that symbol its code. Because the two least-frequent items always get merged (and therefore pushed deeper) first, frequent symbols end up near the root with short codes, and rare symbols end up deep with long codes — and because every symbol lives at a *leaf*, no code is ever a prefix of another, so a decoder can read the bitstream one bit at a time and know exactly when one symbol's code ends and the next begins. This is the core idea behind formats like DEFLATE (used in ZIP and gzip) and was one of the first provably optimal prefix-free compression schemes.
