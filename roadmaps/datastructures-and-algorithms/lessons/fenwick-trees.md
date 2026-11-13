---
title: "Fenwick Trees (Binary Indexed Trees)"
description: A single bit trick, i & -i, turns a plain array into a structure that does prefix sums and point updates in O(log n) with a fraction of a segment tree's code.
roadmap: datastructures-and-algorithms
node: fenwick-trees
group: priority-structures-and-range-queries
video:
  provider: youtube
  id: "RgITNht_f4Q"
  start: 0
  end: null
creator:
  name: WilliamFiset
  channel_url: https://www.youtube.com/@WilliamFiset-videos
duration_seconds: 825
level: intermediate
prerequisites:
  - segment-trees
takeaways:
  - "A Fenwick tree (binary indexed tree, BIT) stores the same n values as the original array but in a cleverly overlapping way, so that both 'update one element' and 'sum a prefix' take O(log n) time using one flat array of size n+1 — no explicit tree nodes or pointers."
  - "Index i (1-based) in the BIT array is responsible for a range of the original array whose length is exactly the value of the lowest set bit of i, which you compute as i & -i using two's-complement arithmetic."
  - "To get the prefix sum up to index i, repeatedly add BIT[i] then subtract the lowest set bit from i (i -= i & -i) and repeat until i reaches 0 — each step jumps to the previous disjoint block, and there are only O(log n) blocks."
  - "To update index i, add the delta to BIT[i] then add the lowest set bit to i (i += i & -i) and repeat until i exceeds n — this walks up to every ancestor range that includes index i."
  - "A Fenwick tree needs roughly a third of the memory and a fraction of the code of a segment tree, because it reuses the original array's indices instead of building a separate tree, and both its core operations are five-line loops."
  - "The i & -i trick only supports operations with an inverse (sums, XORs) because range(l, r) is computed as prefix(r) - prefix(l-1); a Fenwick tree cannot directly answer range-minimum or range-maximum queries, since there's no way to 'subtract' a minimum, which is exactly what a segment tree is for."
check_questions:
  - q: "Why does i & -i give the length of the range that BIT index i is responsible for?"
    a: "In two's-complement, -i flips all bits of i and adds 1, which has the effect that i & -i isolates i's lowest set bit — for example 12 (1100) & -12 gives 4 (0100). That isolated bit's value is exactly the size of the block index i covers: index 12 covers a range of length 4 (indices 9-12), because the binary representation of the index directly encodes how the ranges nest."
  - q: "Why does subtracting i & -i during a prefix-sum query always terminate in O(log n) steps?"
    a: "Each subtraction clears the lowest set bit of i, so the number of steps equals the number of 1-bits in i's binary representation, which is at most log2(n). Every step also moves to a strictly smaller, non-overlapping range, so the sum of BIT[i] values visited exactly reconstructs the prefix without double-counting any element."
  - q: "You need range-minimum queries. Why can't you just build a Fenwick tree the same way you built one for sums?"
    a: "Fenwick trees answer range(l, r) by computing prefix(r) and prefix(l-1) separately and combining them — for sums that's subtraction, which works because addition has an inverse. Minimum has no inverse: knowing min(1, r) and min(1, l-1) doesn't let you recover min(l, r), because the overall minimum could have come from either sub-range. A segment tree avoids this problem because it never decomposes a query into a subtracted pair of prefixes — it directly combines only the nodes inside the query range."
deeper:
  - label: "Fenwick Tree or Binary Indexed Tree (Tushar Roy, alternate walkthrough)"
    url: "https://www.youtube.com/watch?v=CWDQJGaN1gY"
related_formulas: []
---

## Same job, radically less structure

A segment tree answers prefix-sum-style range queries in O(log n), but it needs an explicit tree (or a 4n-sized array standing in for one) and separate code paths for build, query, and update. A Fenwick tree, also called a binary indexed tree (BIT), answers the common special case — prefix sums with point updates — using nothing but a single array of size n+1 and two five-line loops. The trick is that BIT[i] doesn't store element i; it stores the sum of a specific range of the original array, where which range is determined entirely by the binary representation of i.

## The bit trick: i & -i

Index a BIT array starting at 1 (not 0 — this matters). For any index i, compute `i & -i` using two's-complement bitwise AND: negating i flips every bit and adds 1, so ANDing with the original isolates the lowest 1-bit. For i = 12 (binary 1100), -12 in two's complement is ...0100, and 1100 & 0100 = 0100 = 4. That result, 4, is both the size of the range BIT[12] is responsible for (indices 9 through 12) and the "jump size" used when walking the structure.

This isn't a coincidence — it falls out of how the ranges are nested. Index 8 (1000) owns an 8-wide range (1-8); index 12 (1100), which has 8's bit plus an extra 4's bit, owns a narrower 4-wide range nested logically after it; index 10 (1010) owns a 2-wide range; index 11 (1011) owns just itself. The binary digits of the index literally encode a decomposition of "everything up to i" into power-of-two chunks.

## Prefix sum and update, as two loops

**Prefix sum up to i**: repeat `sum += BIT[i]; i -= i & -i;` until i reaches 0. Each subtraction clears the lowest set bit, moving to the next lower, non-overlapping chunk — at most log2(n) iterations, since that's the maximum number of 1-bits in an n-bit number.

**Point update at i (add delta)**: repeat `BIT[i] += delta; i += i & -i;` until i exceeds n. Each addition climbs to the next ancestor range that includes index i — again at most O(log n) steps, since you climb through at most log2(n) "owning" ranges before passing n.

## Worked example

Array (1-indexed): A = [3, 2, -1, 6, 5, 4, -3, 3] for indices 1-8. Each BIT slot holds the sum of the `lowbit(i)`-wide block ending at i: BIT[1]=A[1]=3, BIT[2]=A[1..2]=5, BIT[3]=A[3]=-1, BIT[4]=A[1..4]=10, BIT[5]=A[5]=5, BIT[6]=A[5..6]=9, BIT[7]=A[7]=-3, BIT[8]=A[1..8]=19.

To query prefix(7): start i=7 (binary 0111, lowest bit 1) → sum = BIT[7] = -3, i becomes 7-1=6. i=6 (0110, lowest bit 2) → sum += BIT[6] = 9, total -3+9=6, i becomes 6-2=4. i=4 (0100, lowest bit 4) → sum += BIT[4]=10, total 16, i becomes 4-4=0 → stop. Three jumps (bounded by log2(8)=3) gave prefix(7) = 16, matching a direct sum of A[1..7] = 3+2-1+6+5+4-3 = 16.

Range sum(l, r) is then simply prefix(r) − prefix(l−1) — which is exactly why this only works for invertible operations like sum or XOR, and why range-minimum or range-maximum queries need a segment tree instead: there's no way to "undo" a minimum the way subtraction undoes addition.
