---
title: "D-ary Heaps and Indexed Priority Queues"
description: Giving a heap more than two children per node trades comparisons for height — and an "indexed" heap adds the one operation Dijkstra actually needs.
roadmap: datastructures-and-algorithms
node: d-ary-heaps-and-indexed-priority-queues
group: priority-structures-and-range-queries
video:
  provider: youtube
  id: "DT8xZ0Uf8wo"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 1320
level: intermediate
prerequisites: [trees-bsts-and-heaps]
takeaways:
  - "A d-ary heap gives every node d children instead of 2, which shrinks the tree's height from log2(n) to logd(n) — fewer levels for an element to rise through on insert or decrease-key."
  - "The tradeoff is sift-down: finding the smallest of d children costs d-1 comparisons instead of 1, so sift-down gets more expensive per level even as there are fewer levels."
  - "Decrease-key-heavy workloads (like Dijkstra, which calls it once per edge relaxation) favor a larger d because decrease-key is dominated by the cheap sift-up, not the pricier sift-down, so the shorter tree wins."
  - "A plain array-backed heap only lets you find an element's position by scanning the whole array, which makes decrease-key O(n); an indexed priority queue fixes this by keeping a reverse map from each element's identity to its current slot in the heap array."
  - "That position map turns decrease-key into O(log n): look up the slot in O(1), update the value in place, then sift up or down from exactly that slot instead of searching for it."
  - "The position map must be kept in sync on every swap the heap performs, not just at insert and remove — a sift-down that swaps two nodes has to update both of their recorded positions."
check_questions:
  - q: "Why does increasing d help a decrease-key-heavy workload like Dijkstra more than a workload dominated by extract-min?"
    a: "Decrease-key only does a sift-up, whose cost is proportional to the tree's height (logd n) with no per-level fanout cost, so a bigger d just shortens the climb. Extract-min does a sift-down, which must scan all d children at every level, so a bigger d adds comparison work that partially offsets the shorter height. Dijkstra calls decrease-key once per edge but extract-min once per vertex, so for sparse graphs the cheap-to-improve operation dominates."
  - q: "Without an indexed priority queue, how would Dijkstra have to simulate a decrease-key, and why is that worse?"
    a: "It would push a second, smaller-priority entry for the same vertex into the heap instead of updating the existing one, and later pop and discard the stale duplicate when it's reached. This works but wastes heap space and extract-min calls proportional to the number of relaxations, and it still needs a way to know an entry is stale (a separate visited/distance check) — the indexed heap avoids all of that by actually mutating the entry in place."
  - q: "Why can't you just binary-search the heap array to find where a given element lives, instead of maintaining a position map?"
    a: "A binary (or d-ary) heap's array satisfies the heap-order property between parent and child, not a full sorted order — sibling subtrees aren't ordered relative to each other, so there's no way to prune a search by comparing against one branch. Finding an arbitrary element still requires an O(n) scan unless you separately track where everything is, which is exactly what the position map does."
deeper:
  - label: "Priority Queue Introduction (binary heap fundamentals, for contrast)"
    url: "https://www.youtube.com/watch?v=wptevk0bshY"
related_formulas: []
---

## Wider trees, cheaper climbs

A binary heap is a d-ary heap with d = 2: each node has at most 2 children, and the array layout places node i's children at 2i+1 and 2i+2. There's nothing special about 2. A d-ary heap places node i's children at d·i+1 through d·i+d, and the same two invariants hold — heap order (a parent is never worse than its children) and a complete, left-filled tree — for any d ≥ 2.

The payoff for larger d is height. A binary heap of n elements has height ⌈log2 n⌉; a 4-ary heap has height ⌈log4 n⌉, roughly half as tall. Since insert and decrease-key cost is proportional to how far an element has to sift *up* — one comparison and possibly one swap per level — a shorter tree means fewer comparisons for those operations. With n = 1,000,000: log2 n ≈ 20 levels, log4 n ≈ 10 levels. Sift-up gets twice as fast.

The cost moves to sift-down. Extract-min removes the root, moves the last element there, and sifts it down by repeatedly swapping with its smallest child. In a binary heap that's 1 comparison per level; in a d-ary heap it's d−1 comparisons per level, because you have to find the minimum of d children before you know which way to go. A 4-ary heap does 3 comparisons per level across half as many levels — net it's usually still a win for extract-min, but the margin shrinks as d grows, and past some point (empirically often d ≈ 4) the per-level comparison cost starts to dominate and extract-min gets *slower* than binary.

So the choice of d is a bet on your workload's mix of operations. Dijkstra's algorithm calls decrease-key once per edge relaxation and extract-min once per vertex dequeue. On a graph with many more edges than vertices — the common case — decrease-key calls vastly outnumber extract-min calls, so a 4-ary (even 8-ary) indexed heap frequently beats binary in practice, even though extract-min is individually a bit slower. This is a textbook example of optimizing for the *dominant* operation rather than the asymptotically-cleanest one.

## Why "indexed"

A plain heap only knows the identity of the element at the root. If Dijkstra relaxes an edge and finds a shorter distance to vertex v, it needs to find v's current entry in the heap and decrease its key — but a heap array gives no way to locate v except scanning all n slots, which makes decrease-key O(n) and defeats the entire point of using a heap.

An indexed priority queue (IPQ) fixes this by keeping two arrays alongside the heap: `position[key]` maps an external key (e.g., a vertex id) to its current index in the heap array, and `inverse[index]` maps an index in the heap array back to the key stored there. Every time the heap swaps two slots — during sift-up, sift-down, insert, or removal — it swaps the corresponding entries in `position` and `inverse` too, so the mapping never goes stale.

With that mapping, decrease-key(v, newDist) becomes: look up `i = position[v]` in O(1), overwrite the value at heap[i], then sift-down *or* sift-up from i (only one direction can apply, since the value only moved in one direction) in O(logd n). No search required.

## Worked example

Take a 4-ary indexed min-heap holding distance estimates for vertices A–F: [A:2, B:9, C:9, D:11, E:12, F:7]. With d=4, A (index 0) has children at indices 1–4: B, C, D, E.

Say an edge relaxation drops F's distance from 7 to 1. Decrease-key looks up `position[F] = 5`, writes 1 at index 5, then sifts up. For a 4-ary heap, `parent(i) = ⌊(i-1)/d⌋`, so index 5's parent is index `⌊4/4⌋ = 1`, which holds B. Since 1 < 9, swap F and B, update both position entries, and continue comparing against A.

F is now at index 1, whose parent is index 0: A, holding 2. Since 1 < 2, swap again. F is now the root, in two swaps total — `O(log_d n)` — with no scan needed to find it.
