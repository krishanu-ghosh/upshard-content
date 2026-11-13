---
title: "Dynamic Arrays and Amortized Analysis"
description: How doubling-based growth turns an occasional O(n) resize into O(1) amortized append, and why shrinking too eagerly destroys that guarantee.
roadmap: datastructures-and-algorithms
node: dynamic-arrays-and-amortized-analysis
group: advanced-linear-structures
video:
  provider: youtube
  id: "MTl8djZFWE0"
  start: 0
  end: null
creator:
  name: Gaurav Sen
  channel_url: https://www.youtube.com/@gkcs
duration_seconds: 760
level: intermediate
prerequisites: [arrays-linked-lists-stacks-and-queues]
takeaways:
  - "A dynamic array is a fixed-size backing array underneath — append only looks unbounded because the array quietly replaces itself with a bigger one when it runs out of room."
  - "Growing by doubling (not by a fixed increment like +1 or +10) is what makes amortized append O(1); growing by a constant amount degrades amortized append to O(n)."
  - "A single append can cost O(n) when it triggers a resize, but because resizes happen on a shrinking fraction of appends, the average cost per append across any long sequence stays O(1)."
  - "Shrinking the backing array as soon as it drops below half full, then regrowing it as soon as one more element is appended, causes thrashing — a repeated resize on almost every operation — which is why real implementations only shrink at a much lower threshold, like a quarter full."
  - "A fixed-size array has no resize logic at all: it gives true O(1) worst-case access but a hard capacity ceiling, trading the dynamic array's amortized guarantee for a stricter, more predictable one."
check_questions:
  - q: "If a single append can cost O(n) whenever it triggers a resize, why do we still call append O(1)?"
    a: "We're describing amortized cost, not worst-case cost for one call. Resizes get exponentially rarer as the array grows — doubling means only about log2(n) resizes happen across n appends total — so the total copying work across any sequence of n appends is O(n), which averages to O(1) per append even though a handful of individual appends are expensive."
  - q: "Why does doubling give O(1) amortized append, but growing the array by a fixed amount (say, always adding 100 slots) does not?"
    a: "With a fixed increment, the capacity grows linearly with the number of appends, so a resize happens roughly every 100 appends regardless of how large the array already is, and each resize still copies all existing elements. Across n appends, that's roughly n/100 resizes each copying up to n elements, which sums to O(n^2) total work — O(n) amortized per append, not O(1). Doubling instead makes resizes exponentially less frequent precisely because the array being copied is also exponentially larger, which is the balance that keeps the total linear."
  - q: "A naive dynamic array shrinks the backing array in half the moment it drops below 50% full. Why is this a bad idea, and what's the usual fix?"
    a: "Imagine an array sitting at exactly half full: one more append triggers a resize up, and then one removal immediately triggers a resize back down — repeat that append/remove pattern and every single operation costs O(n), destroying the amortized O(1) guarantee entirely. This is called thrashing. The standard fix is to use different thresholds for growing and shrinking — for example, double when 100% full but only shrink when down to 25% full (not 50%) — so there's a wide buffer zone where neither direction triggers a resize."
deeper:
  - label: "Dynamic Array Amortized Analysis (Interview Cake)"
    url: "https://www.interviewcake.com/concept/dynamic-array-amortized-analysis"
related_formulas: []
---

## The array underneath the array

A Python `list`, Java `ArrayList`, and C++ `vector` all look like they grow without limit, but underneath, each one is backed by a plain fixed-size array. The dynamic array type is really just a wrapper around that fixed array plus two numbers: `size` (how many elements are actually stored) and `capacity` (how many slots the backing array has). Appending normally means "write the new element at index `size`, then increment `size`" — a single array write, O(1).

The interesting case is what happens when `size` reaches `capacity`. There's no room left, so the wrapper has to: allocate a brand new backing array (conventionally twice the old capacity), copy every existing element from the old array into the new one, free the old array, and only then write the new element. That copy touches every element currently stored, so it costs O(n) where n is the current size — a single append can be dramatically more expensive than the O(1) case a moment earlier.

## Why doubling makes the average work out

This is where amortized analysis earns its keep: it asks about the total cost of a long sequence of operations, divided by the number of operations, rather than the cost of any one operation in isolation.

Walk through appending n elements one at a time, starting from an empty array. Resizes happen when the array holds 1, 2, 4, 8, 16, ... elements — each time the capacity doubles. The cost of each resize equals the array's size at that point, since that's how many elements get copied. So the total copying work across all resizes up to n elements is roughly:

1 + 2 + 4 + 8 + ... + n/2 + n

This is a geometric series, and geometric series have a convenient property: the sum is bounded by about 2n — less than twice the final term. So the total work spent on *all* resizing during n appends is O(n), not O(n log n) or O(n²). Divide that O(n) total by the n appends that produced it, and the amortized cost per append is O(n)/n = O(1). Most appends cost one array write; the rare expensive ones are so infrequent, and shrink so fast relative to how large the array has become, that their total contribution averages out to a constant.

Contrast this with growing by a fixed increment, like adding a flat 100 slots every time the array fills up. Now a resize happens roughly every 100 appends no matter how big the array already is, and each resize still copies up to n elements if the array has grown to size n. Across n appends, that's about n/100 resizes, each costing up to O(n) — a total of O(n²), or O(n) amortized per append. The reason doubling works and fixed growth doesn't is that doubling ties the *frequency* of expensive resizes to the *size* of the array being copied: by the time a resize is expensive, it's also become correspondingly rare.

## The other direction: shrink-and-grow thrashing

The same logic has to be applied carefully in reverse when elements are removed. A tempting rule is "shrink the backing array to half its capacity whenever the array drops below 50% full." This looks symmetric with growing at 100% full, but it's actually broken.

Walk it through with numbers. Capacity 8, size 4 — right at 50%, stable. Remove one element: size 3, which is below 50% of 8, so it triggers a shrink to capacity 4. Size 3 is now 75% of the new capacity. Append one element: size 4, which is 100% of capacity 4, triggering a grow back to capacity 8. Remove one element: size 3, which is below 50% of 8 again — another shrink.

An adversarial (or just unlucky) sequence of alternating appends and removals near that boundary forces a full resize on *every single operation*, which is O(n) per operation. The amortized guarantee is gone entirely. This failure mode is called thrashing.

The standard fix is to make the grow and shrink thresholds asymmetric, leaving a buffer zone where neither triggers. A common scheme: double the capacity when 100% full, but only shrink (typically by half) when usage drops to 25% full, not 50%. Now the array has to lose three-quarters of its elements before a shrink fires, and immediately afterward it sits at 50% full — comfortably inside the no-resize zone on both sides. The amortized analysis for removals mirrors the doubling argument for appends exactly, as long as the thresholds keep that gap.

## Fixed arrays as the alternative

A plain fixed-size array never does any of this: every access is O(1) worst case, not just amortized, because there's no resize logic to trigger. The cost is a hard ceiling — once it's full, there's no way to add another element without building an entirely new, larger structure yourself. Dynamic arrays trade that worst-case guarantee for flexibility: any individual append *might* be slow, but you're buying the convenience of never having to think about capacity, at the cost of an occasional, bounded, and statistically rare spike.
