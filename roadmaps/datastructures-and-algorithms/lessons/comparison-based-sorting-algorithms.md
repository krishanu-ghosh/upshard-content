---
title: "Comparison-Based Sorting: Six Algorithms, Two Shapes"
description: Bubble, selection, insertion, merge, quick, and heap sort compared on time complexity, space, stability, and when each one actually wins.
roadmap: datastructures-and-algorithms
node: comparison-based-sorting-algorithms
group: sorting-searching-and-recursion
video:
  provider: youtube
  id: "gcRUIO-8r3U"
  start: 0
  end: null
creator:
  name: Greg Hogg
  channel_url: https://www.youtube.com/@GregHogg
duration_seconds: 2500
level: beginner
prerequisites:
  - arrays-linked-lists-stacks-and-queues
takeaways:
  - Bubble, selection, and insertion sort are all O(n²) in the worst case because each one does a nested pass over the remaining unsorted elements, but insertion sort is the only one of the three that's fast on data that's already nearly sorted.
  - Merge sort guarantees O(n log n) in every case by splitting the array in half recursively and then merging two sorted halves, at the cost of O(n) extra memory for the merge buffer.
  - Quick sort is O(n log n) on average by partitioning around a pivot, but a bad pivot choice on already-sorted or adversarial input degrades it to O(n²), which is why production implementations randomize the pivot or pick the median of three.
  - Heap sort turns an array into a max-heap and then repeatedly swaps the root with the last unsorted element and sifts down, giving O(n log n) worst-case time with O(1) extra space — the one sort here that's both guaranteed-fast and in-place.
  - Stability — whether equal elements keep their original relative order — matters whenever you sort by one key after already sorting by another; insertion sort and merge sort are stable, while selection sort, quick sort, and the usual array-based heap sort are not.
check_questions:
  - q: Why does insertion sort outperform bubble and selection sort on data that's already almost sorted?
    a: Insertion sort only shifts an element as far as it needs to go to reach its correct position, so on nearly-sorted input most elements need zero or one shift, making a full pass closer to O(n) than O(n²); bubble and selection sort always scan the full remaining range regardless of how sorted it already is.
  - q: What's the practical risk of picking the first element as a quick sort pivot, and how do real implementations avoid it?
    a: On input that's already sorted (or reverse-sorted), the first element is always the smallest or largest possible pivot, so every partition splits into a size-0 and a size-(n-1) group, degrading quick sort to O(n²); real implementations instead randomize the pivot or use median-of-three to make that worst case extremely unlikely.
  - q: Why is heap sort described as "guaranteed-fast and in-place" compared to merge sort and quick sort?
    a: Heap sort's worst-case time is O(n log n) like merge sort's, but unlike merge sort it needs no extra array for merging, and unlike quick sort its worst case never degrades to O(n²), so it's the one algorithm here that combines a reliable time bound with O(1) extra space.
deeper:
  - label: "Sorting Algorithms, Visualized"
    url: "https://www.youtube.com/watch?v=7RlWhw8D-9A"
related_formulas: []
---

## Two families, one goal

Every comparison-based sort answers the same question: put these elements in order, by comparing pairs of elements and deciding which comes first. The six classics split into two families by how fast they scale.

Bubble, selection, and insertion sort are all O(n²) in the worst case, doing roughly n passes over roughly n elements each. Merge sort, quick sort, and heap sort are all O(n log n), because each one finds a way to throw away more than one element's worth of work per comparison: recursively halving the problem (merge sort), partitioning around a pivot (quick sort), or exploiting the structure of a heap (heap sort).

The n² sorts are simpler to write and fine for small or nearly-sorted inputs. The n log n sorts are what every standard library actually uses once n gets large.

## The O(n²) family

**Bubble sort** repeatedly walks the array, swapping any adjacent pair that's out of order. After each full pass, the largest remaining element has "bubbled" to its final position.

**Selection sort** instead scans the unsorted remainder to find the minimum and swaps it into place at the front. It does the same number of comparisons as bubble sort but far fewer swaps, which matters when writes are expensive (flash memory, for instance).

**Insertion sort** builds up a sorted prefix one element at a time, inserting each new element into the position it belongs in among the elements already placed. Think of sorting a hand of playing cards by picking up one card at a time.

All three are O(n²) worst case, because the outer loop runs n times and the inner work is O(n) each time. But insertion sort's inner loop only runs as far as it needs to. On data that's already sorted, insertion sort is O(n), which is why it's often the fallback for small subarrays inside a faster sort.

## Merge sort: divide, conquer, merge

Merge sort splits the array in half, recursively sorts each half, and then merges the two sorted halves back together in one linear pass. At each step, compare the fronts of the two halves and take the smaller one.

Splitting in half happens log n times, and merging at each level touches all n elements once. So the total work is O(n log n) in every case, not just on average. There's no input that makes merge sort degrade.

The cost is space. Merging two sorted arrays in place is awkward, so the standard implementation allocates an auxiliary array of size n, giving merge sort O(n) extra memory. It's also stable: when two elements compare equal, the merge step always takes from the left half first, so their original order survives.

## Quick sort: partition around a pivot

Quick sort picks a pivot element, then partitions the array so everything less than the pivot ends up on one side and everything greater ends up on the other, with the pivot itself landing in its final sorted position. It then recurses on the two sides.

When the pivot consistently splits the array roughly in half, that's the same log n levels of recursion as merge sort, giving O(n log n) average time. Quick sort does this in place, with no auxiliary array, which is why it's usually faster in practice despite having the same big-O as merge sort.

The catch is the worst case. A naive "always pick the first element" pivot, run on input that's already sorted, is always the smallest or largest element in its partition. Each partition then only shrinks the problem by one, and the recursion depth becomes n — giving O(n²). Randomizing the pivot choice (or picking the median of the first, middle, and last elements) makes that worst case vanishingly unlikely on real-world input, which is why every production quick sort does one of the two.

## Heap sort: the structure does the work

A binary heap is a tree stored in an array where every parent is at least as large as its children (a max-heap). Heap sort first turns the input array into a max-heap in place — the "heapify" step, which runs in O(n). It then repeatedly takes the root (always the current maximum), swaps it with the last unsorted element, shrinks the heap by one, and sifts the new root down to restore the heap property.

Each of the n extraction-and-sift-down steps costs O(log n), for O(n log n) total. Every operation happens by swapping elements within the original array, so heap sort needs no extra memory beyond a few variables: O(1) extra space.

That combination is unique among the six. Merge sort pays for its O(n log n) guarantee with O(n) space. Quick sort pays for its O(1) space with a possible O(n²) worst case. Heap sort gets the guarantee and the space, but it isn't stable (the heap operations reorder equal elements), and its constant factors tend to be worse in practice than a well-tuned quick sort — swapping elements up and down a heap touches memory less predictably than quick sort's sequential partitioning scan.

## Choosing one

In practice, almost nobody hand-picks a single algorithm for every case. Python's `sort()` and Java's `Arrays.sort()` for objects use Timsort: insertion sort for small runs, merged together like merge sort. Insertion sort's near-linear behavior on small or partially-sorted data is a real win that pure merge sort or quick sort would throw away.

Primitive arrays in Java and C++'s `std::sort` typically use an introspective quick sort that falls back to heap sort if the recursion gets too deep. That's exactly how engineers cover quick sort's worst case without giving up its usual speed.

Knowing all six is less about picking a favorite than about recognizing which tradeoff actually matters for the data in front of you: guaranteed time, memory, stability, or real-world constant factors.
