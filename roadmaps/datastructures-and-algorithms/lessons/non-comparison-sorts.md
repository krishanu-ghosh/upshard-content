---
title: "Non-Comparison Sorts: Counting, Radix, and Bucket Sort"
description: How counting, radix, and bucket sort break the Ω(n log n) barrier by never comparing elements, and when that trick applies.
roadmap: datastructures-and-algorithms
node: non-comparison-sorts
group: advanced-sorting-and-selection
video:
  provider: youtube
  id: "pJ1IQD5rv4o"
  start: 0
  end: null
creator:
  name: Algorithms with Attitude
  channel_url: https://www.youtube.com/@AlgorithmswithAttitude
duration_seconds: 1184
level: intermediate
prerequisites: [comparison-based-sorting-algorithms]
takeaways:
  - "The Ω(n log n) lower bound only applies to comparison sorts: it's a limit on how much information a sequence of 'is A before B?' questions can extract, and counting, radix, and bucket sort sidestep it by using the actual values of the keys instead of comparing them."
  - "Counting sort works by tallying how many times each key value appears in a count array, then turning those counts into prefix sums that give each element's final position directly, in O(n + k) time where k is the range of key values."
  - "Counting sort is only practical when k, the range of possible key values, is not much larger than n — sorting a million integers from 0 to 100 is O(n + k) ≈ O(n), but sorting them from 0 to 10 billion is O(n + k) ≈ O(k), which is worse than any comparison sort."
  - "Radix sort gets around a huge key range by sorting digit-by-digit instead of all-digits-at-once, running a stable counting sort on each digit from least-significant to most-significant, for a total cost of O(d(n + k)) where d is the number of digits and k is the base (usually 10 or 256)."
  - "Radix sort's correctness depends entirely on each per-digit pass being stable — if two elements with different values at a lower digit got swapped out of order by an unstable pass, a later, higher-digit pass could never fix it."
  - "Bucket sort handles real-valued, roughly uniformly distributed keys by scattering elements into n buckets by value range, sorting each small bucket with any algorithm (often insertion sort), and concatenating — it's O(n) on average but degrades toward O(n²) if the data clusters into one bucket instead of spreading out."
check_questions:
  - q: "Why doesn't the Ω(n log n) comparison-sort lower bound apply to counting sort?"
    a: "The lower bound is a statement about decision trees built from pairwise comparisons: with only 'less than' or 'greater than' answers, you need log(n!) ≈ n log n comparisons to distinguish all n! orderings. Counting sort never asks 'is A less than B?' at all — it reads each key's value directly and uses it as an index into a count array, so the lower bound's assumptions simply don't apply to it."
  - q: "You need to sort 10 million 64-bit integers that are spread across the full range of possible 64-bit values. Why is counting sort a bad choice here, and what does radix sort do differently?"
    a: "Counting sort's cost is O(n + k), and here k (the range of possible values) is roughly 2^64 — astronomically larger than n, so the count array alone would be infeasible. Radix sort instead treats each number as d digits in some base (e.g. 8 bytes in base 256) and runs a cheap counting sort on each digit separately, so its cost is O(d(n + k)) with k now just 256 and d = 8 — linear in n instead of dominated by the key range."
  - q: "If you bucket-sort an array of a million floats that happen to all fall between 0.0 and 0.001, what goes wrong, and why?"
    a: "Bucket sort assumes keys are roughly uniformly spread across the full range, so it can map value ranges to buckets and expect each bucket to get about n/(number of buckets) elements. If every value is actually crammed into a tiny sub-range, the bucket-assignment formula puts almost everything into one (or a few) buckets, and sorting that one overloaded bucket with insertion sort costs close to O(n²) — the 'average case O(n)' guarantee relies on the distribution assumption holding."
deeper:
  - label: "Linear-time sorting, part 2: Counting Sort, Radix Sort and Bucket Sort"
    url: "https://www.youtube.com/watch?v=CJzT3fMy0h4"
related_formulas: []
---

## Why "no comparisons" changes everything

Every sort covered in `comparison-based-sorting-algorithms` — quicksort, mergesort, heapsort, the O(n²) trio — works by asking a sequence of "is this element less than that one?" questions and using the answers to build an ordering. That restriction is exactly what produces the Ω(n log n) lower bound: with n elements there are n! possible orderings, and each comparison only narrows things down by a factor of 2, so you need at least log₂(n!) ≈ n log n comparisons in the worst case to figure out which ordering you've got, no matter how cleverly you pick which pairs to compare. This is a hard limit on comparison-based algorithms specifically — it says nothing about algorithms that don't compare elements pairwise at all.

Counting sort, radix sort, and bucket sort escape the bound by using extra structure in the keys: they read a key's actual value and use it to compute a position directly, which carries far more information per key than a single "less than" answer ever could. The tradeoff is that they only work when the keys have that exploitable structure — bounded integers, or values spread uniformly over a known range. Hand them arbitrary comparable objects (strings of unknown length, custom objects with an overloaded `<`) and they either don't apply or degrade badly.

## Counting sort: tally, then place

Counting sort assumes keys are integers in a known range `[0, k)`. It makes two passes. First, build a count array `C` of size k where `C[v]` is the number of elements equal to v — one O(n) pass over the input, incrementing counts. Second, turn `C` into a prefix-sum array, so `C[v]` now holds how many elements are ≤ v; that number is exactly the final index (plus one) of the last occurrence of value v in the sorted output. Walking the original array once more (from the end, to keep the sort stable) and placing each element at position `C[value] - 1`, then decrementing `C[value]`, produces the fully sorted array. Total cost: O(n) to build the histogram, O(k) to prefix-sum it, O(n) to place elements — O(n + k) overall, with no comparisons anywhere.

Worked example: sort `[4, 1, 3, 4, 1]` with k = 5 (values 0–4). Counts: `C = [0, 2, 0, 1, 2]`. Prefix sums: `C = [0, 2, 2, 3, 5]`. Now place from the right: the second `1` (index 4) goes to position `C[1] - 1 = 1`, decrement `C[1]` to `1`. The `4` (index 3) goes to position `C[4] - 1 = 4`, decrement `C[4]` to `4`. The `3` (index 2) goes to position `C[3] - 1 = 2`, decrement `C[3]` to `2`. The `1` (index 1) goes to position `C[1] - 1 = 0`, decrement `C[1]` to `0`. The `4` (index 0) goes to position `C[4] - 1 = 3`, decrement `C[4]` to `3`. Result: `[1, 1, 3, 4, 4]`. The catch is k: if keys range over billions of values, the count array itself becomes the bottleneck.

## Radix sort: counting sort, one digit at a time

Radix sort handles keys with a huge range by never building a histogram over the whole range at once. Instead, treat each key as a sequence of d digits in some base (base 10 for human-readable digits, base 256 for byte-at-a-time on binary integers), and run a stable counting sort on each digit position, going from the **least significant digit to the most significant**. Stability is what makes this work: after sorting by the last digit, elements that tie on the last digit keep their relative order; sorting that result by the second-to-last digit then only needs to "break ties" correctly among groups that already agree on lower digits, and so on up to the most significant digit, after which the whole array is fully sorted.

Worked example: sort `[170, 45, 75, 90, 802, 24, 2, 66]` (3 digits, base 10). Pass on the 1s digit groups them by last digit: `[170, 90, 802, 2, 24, 45, 75, 66]`. Pass on the 10s digit: `[802, 2, 24, 45, 66, 170, 75, 90]`. Pass on the 100s digit: `[2, 24, 45, 66, 75, 90, 170, 802]` — sorted. Each digit pass costs O(n + k) (k = 10 here), and there are d passes, so total cost is O(d(n + k)). When d is a small constant (fixed-width integers), this is effectively O(n).

## Bucket sort: scatter, sort small, gather

Bucket sort targets real-valued keys assumed to be roughly uniformly distributed over a known range, like `[0.0, 1.0)`. Create n empty buckets, and map each key `x` to bucket `floor(x * n)`, scattering elements across the buckets in one O(n) pass. If the distribution really is uniform, each bucket ends up with O(1) elements on average, so sorting each bucket individually — with insertion sort, since small near-sorted lists are cheap — costs O(1) per bucket, O(n) total. Concatenating the sorted buckets in order gives the fully sorted array. Expected time: O(n). The assumption is load-bearing: if the data isn't uniform and clusters into a few buckets, those buckets degrade toward sorting n elements with insertion sort, i.e. O(n²).

## Choosing among them

None of these three replace quicksort or mergesort as general-purpose sorts — they apply only when the keys have known, exploitable structure. Counting sort is the right call when keys are small-range integers (sorting grades 0–100, or a small finite set of categories). Radix sort extends that to fixed-width integers or strings with a huge value range but few digits (sorting 32-bit IDs, fixed-length strings). Bucket sort is the pick when keys are real numbers known to be roughly uniform over a range (sorting normalized scores, timestamps within a known window). Outside those conditions — arbitrary objects, unknown or skewed distributions, variable-length keys with no digit structure — comparison sorts remain the safer general tool, because their guarantees don't depend on an assumption about the input that might be wrong.
