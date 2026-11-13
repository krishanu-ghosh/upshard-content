---
title: "Sparse Tables and Static Range Queries"
description: Answer range-minimum queries in O(1) on a never-updated array using a sparse table built in O(n log n) from overlapping power-of-two ranges.
roadmap: datastructures-and-algorithms
node: sparse-tables-and-static-range-queries
group: more-advanced-structures
video:
  provider: youtube
  id: "uUatD9AudXo"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 1398
level: intermediate
prerequisites: ["segment-trees"]
takeaways:
  - "A sparse table precomputes the answer for every range whose length is a power of two, storing table[k][i] as the answer (e.g. minimum) over the range starting at i with length 2^k."
  - "Building the table takes O(n log n) time and space: for each of the O(log n) power-of-two lengths, every starting position's answer is computed in O(1) from two already-computed smaller ranges."
  - "Any query range can be covered by two overlapping power-of-two-length ranges that together span it completely, which is why queries run in O(1) — no loop over the range is needed."
  - "That overlap trick only works for idempotent operations like min, max, and gcd, where covering part of the range twice doesn't corrupt the answer — it does not work for sum, where double-counting overlapping elements gives a wrong total."
  - "A sparse table cannot handle updates: changing one array element would invalidate every precomputed range that includes it, which is O(n log n) worth of entries — exactly the operation a segment tree is built to handle in O(log n) instead, at the cost of slower queries."
check_questions:
  - q: "Why can any query range [L, R] be answered using only two precomputed ranges, instead of needing to combine many power-of-two pieces?"
    a: "For min/max/gcd-style idempotent queries, covering part of the range more than once doesn't change the answer — the minimum of a multiset is unaffected by counting some elements twice. So instead of tiling the range with several non-overlapping power-of-two pieces (which a Fenwick-tree-style decomposition would need), you can pick the single largest power of two, 2^k, that fits inside the range's length, and take one range of that length starting at L and another of that same length ending at R. Together they're guaranteed to cover every index in [L, R], possibly overlapping in the middle, and since overlap is harmless, combining just those two precomputed answers gives the correct result in O(1)."
  - q: "Why doesn't the sparse table's O(1)-query trick work for range sum queries?"
    a: "Sum is not idempotent: if the two overlapping precomputed ranges both include some of the same elements, those elements get added into the total twice, inflating the sum. The two-range overlap trick relies on the combining operation being insensitive to repeated inclusion of the same element, which holds for min, max, and gcd but not for sum (or product, or count) — those need the ranges to be non-overlapping, which is a different, more involved decomposition."
  - q: "An array needs occasional single-element updates along with frequent range-minimum queries. Why would a segment tree be the better choice here even though its queries are slower than a sparse table's?"
    a: "A sparse table's O(1) query comes from precomputing answers for every power-of-two range in O(n log n) once, up front; changing a single array element would invalidate every precomputed range that happens to include that index, which is O(n log n) worth of stale entries to fix — effectively forcing a full rebuild. A segment tree keeps query time at O(log n) instead of O(1), but it supports updating a single element in O(log n) by only recomputing the O(log n) ancestor nodes on the path to the root. Once updates are part of the workload, the sparse table's one-time preprocessing assumption is broken, and the segment tree's balanced tradeoff between update and query cost wins."
deeper:
  - label: "cp-algorithms: Sparse Table"
    url: "https://cp-algorithms.com/data_structures/sparse-table.html"
related_formulas: []
---

## The setup: queries on an array that never changes

A segment tree answers range queries in O(log n) and also supports point or range updates in O(log n) — it's a general-purpose tool. But a large class of problems never update the array at all: the array is built once and then queried many times, often for the minimum or maximum over a range. When updates are off the table entirely, a sparse table can push query time all the way down to O(1), at the cost of a heavier one-time preprocessing step and the inability to handle any update whatsoever.

## Precomputing every power-of-two range

The structure stores a 2D table, commonly written `table[k][i]`, where `table[k][i]` holds the answer (say, the minimum) over the range of length `2^k` starting at index `i`: `[i, i + 2^k - 1]`. There are O(log n) useful values of `k` (since `2^k` can't exceed `n`), and O(n) starting positions `i` for each, giving O(n log n) total entries.

The key to building this efficiently is that a range of length `2^k` splits perfectly in half into two ranges of length `2^(k-1)`, both already computed in the previous layer: `table[k][i] = min(table[k-1][i], table[k-1][i + 2^(k-1)])`. So each layer is built in O(n) from the one below it, using only already-known values — O(1) work per entry — and with O(log n) layers total, the whole table costs O(n log n) time and space to build. The base case, `table[0][i]`, is just the array's own value at `i` (a range of length `2^0 = 1`).

## Answering a query in O(1)

To answer "what's the minimum in `[L, R]`?", let `len = R - L + 1` and let `k = floor(log2(len))` — the largest power of two that fits inside the range. Two precomputed ranges of length `2^k` are guaranteed to cover `[L, R]` completely: one starting at `L` (covering `[L, L + 2^k - 1]`), and one ending at `R` (covering `[R - 2^k + 1, R]`). Together they span every index in `[L, R]`, though they may overlap in the middle if `2^k` is less than half of `len`. The answer is simply `min(table[k][L], table[k][R - 2^k + 1])` — two array lookups and one comparison, regardless of how large the range is. That's the entire query: no loop, no tree descent.

## Worked example

Array: `[5, 2, 4, 7, 1, 3, 6, 8]` (indices 0–7). Build the base layer `table[0][i] = arr[i]` for all `i`. Then `table[1][i] = min(table[0][i], table[0][i+1])`, giving ranges of length 2: `table[1] = [2, 2, 4, 1, 1, 3, 6]` (e.g. `table[1][0] = min(5,2) = 2`). Then `table[2][i] = min(table[1][i], table[1][i+2])`, ranges of length 4: `table[2][0] = min(table[1][0], table[1][2]) = min(2, 4) = 2`, and so on.

Query `[1, 6]` (values `2,4,7,1,3,6`, true minimum 1): `len = 6`, so `k = floor(log2(6)) = 2`, meaning we use ranges of length `2^2 = 4`. One range starts at `L=1`: `table[2][1]`, covering indices 1–4. The other ends at `R=6`, so it starts at `R - 2^k + 1 = 6 - 4 + 1 = 3`: `table[2][3]`, covering indices 3–6. These overlap at indices 3–4, but since we're taking a minimum, double-covering those indices changes nothing. `table[2][1] = min(2,4,7,1) = 1` and `table[2][3] = min(7,1,3,6) = 1`. Final answer: `min(1, 1) = 1` — correct, in two lookups.

## Why it can't handle updates, and why that's the real tradeoff

That overlap trick only works because minimum (and maximum, and gcd) is **idempotent** — including an element twice doesn't change the result. It would silently break for range *sum*, where overlapping ranges would double-count shared elements and produce the wrong total; sum queries need a different, non-overlapping decomposition entirely (which is part of why Fenwick trees exist as a separate structure for sums).

The bigger limitation is updates. If a single array element changes, every precomputed range in the table that includes that index is now stale — and because ranges of every power-of-two length starting near that index are affected, that's O(n log n) entries that would need recomputation, effectively a full rebuild. This is the sparse table's entire tradeoff in one sentence: it trades away any ability to update, in exchange for genuinely O(1) queries, which is exactly the opposite tradeoff a segment tree makes (O(log n) for both query and update). Choosing between them comes down to one question: does the array ever change after it's built? If yes, reach for a segment tree; if the array is truly static and only queried, the sparse table's O(1) queries are hard to beat.
