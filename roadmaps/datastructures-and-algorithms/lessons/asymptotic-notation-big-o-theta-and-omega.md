---
title: "Asymptotic Notation: Big-O, Big-Theta, and Big-Omega"
description: How Big-O, Big-Theta, and Big-Omega describe an algorithm's growth rate, and how to work out which one applies to your own code.
roadmap: datastructures-and-algorithms
node: asymptotic-notation-big-o-theta-and-omega
group: complexity-and-math-foundations
video:
  provider: youtube
  id: "A03oI0znAoc"
  start: 0
  end: null
creator:
  name: Abdul Bari
  channel_url: https://www.youtube.com/@abdul_bari
duration_seconds: 840
level: beginner
prerequisites: []
takeaways:
  - "Asymptotic notation describes how an algorithm's running time or memory use grows as the input size grows, not how fast it runs on one particular machine or input."
  - "Big-O gives an upper bound on growth — \"this algorithm never does worse than this\" — and is the notation you'll see used loosely (and often imprecisely) as a synonym for \"the complexity\"."
  - "Big-Omega gives a lower bound — \"this algorithm never does better than this\" — which matters when you want to prove no algorithm for a problem can be faster than a certain rate."
  - "Big-Theta applies only when the upper and lower bounds meet, giving a tight description of growth rather than just a ceiling or a floor."
  - "Dropping constants and lower-order terms is deliberate: f(n) = 3n² + 5n + 2 and f(n) = n² behave the same way once n is large enough, and that's the regime these notations are built to describe."
check_questions:
  - q: Why does Big-O analysis ignore constant factors like the "3" in 3n²?
    a: Because asymptotic notation is about the long-run shape of growth as n gets large, and a constant factor never changes whether that shape is linear, quadratic, or exponential — it only rescales the graph, which matters for real-world speed but not for classifying the algorithm.
  - q: An algorithm takes between n and n² steps depending on the input, and in the worst case it genuinely does take close to n² steps. What's the tightest single-notation description of its worst case?
    a: O(n²) for the worst case specifically, since Big-O is an upper bound and n² is reached; whether it's Theta(n²) in the worst case too depends on whether that bound is actually met and not just approached, but across all inputs no single Theta bound would apply since the running time varies between n and n².
  - q: What's the difference between saying an algorithm "is O(n log n)" and saying it "is Theta(n log n)"?
    a: O(n log n) only promises the algorithm won't do worse than n log n — it could secretly always run in O(n) time and that statement would still be true; Theta(n log n) is a stronger, tighter claim that the algorithm's growth rate is bounded both above and below by n log n, so it genuinely grows at that rate, not just "at most" that rate.
deeper:
  - label: "Introduction to Big O Notation and Time Complexity"
    url: "https://www.youtube.com/watch?v=D6xkbGLQesk"
related_formulas: []
---


## Why "how many steps" isn't a useful question

Ask "how long does this algorithm take?" and the honest answer is "it depends" — on the input size, on the specific input, on the CPU, on the language, on whether the cache is warm. None of that is useful for comparing two algorithms in the abstract. What *is* useful is a much narrower question: as the input gets bigger and bigger, how does the cost grow? Does doubling the input double the work, square it, or barely move it? That question has a clean answer independent of hardware, and asymptotic notation is the vocabulary for stating it.

The "asymptotic" part means we care about behavior as the input size n approaches infinity, not about small n where constant overhead and one-off setup costs dominate. A method that takes `1000 + n` steps and one that takes `n` steps are asymptotically identical, even though the first is slower for every n you'd ever actually run — the 1000 is a constant, and constants wash out once n is large enough.

## Big-O: an upper bound

Big-O notation, written O(f(n)), states that a function's growth is bounded above by f(n) for all sufficiently large n, up to a constant factor. Formally: g(n) is O(f(n)) if there exist constants c > 0 and n₀ such that g(n) ≤ c·f(n) for every n ≥ n₀. In plain terms: "this never gets worse than a constant multiple of f(n), once n is big enough."

Take linear search through an unsorted array of n elements: in the worst case — the target is last, or absent — it checks every element, so its running time is O(n). Binary search on a sorted array halves the remaining range each step, so it takes O(log n): you can double the array size and only add one more comparison. Bubble sort compares and possibly swaps pairs in nested loops, giving O(n²): double the input and the work roughly quadruples.

Because Big-O is an *upper* bound, it's technically correct — if unhelpfully loose — to say linear search is O(n²), since n² is certainly an upper bound on n. In practice, people use O(f(n)) to mean the *tightest* upper bound they can state, and that convention is why you'll hear "it's O(n)" used as shorthand for "its growth rate is n," even though that's really a Big-Theta claim (see below).

## Big-Omega: a lower bound

Big-Omega, written Ω(f(n)), is the mirror image: it bounds growth from *below*. g(n) is Ω(f(n)) if there exist constants c > 0 and n₀ such that g(n) ≥ c·f(n) for all n ≥ n₀ — "this never does better than a constant multiple of f(n)."

Lower bounds are most useful for a whole *problem*, not just one algorithm's implementation. Comparison-based sorting is Ω(n log n): no algorithm that sorts purely by comparing pairs of elements can do better than n log n comparisons in the worst case, which is a fact you can prove by counting how many possible orderings n items have and noting each comparison only rules out some of them. That's why merge sort and heap sort, both O(n log n), are considered asymptotically optimal comparison sorts — you can't beat the Ω(n log n) floor for this class of algorithm, though non-comparison sorts like counting sort can, under the right conditions, since they don't rely on pairwise comparisons at all.

## Big-Theta: when the bounds meet

Big-Theta, written Θ(f(n)), applies when a function is both O(f(n)) and Ω(f(n)) — the growth rate is pinned down from above and below by the same f(n), up to constants. This is the tightest and most informative of the three: it says the algorithm genuinely grows at rate f(n), not merely "at most" or "at least."

Binary search is Θ(log n): it's never slower than c₁·log n (the upper bound) and, because it genuinely has to narrow down among n possibilities, it's never faster than c₂·log n either (the lower bound). Linear search's worst case is Θ(n) for the same reason — in the worst case it really does have to look at every element, so n isn't just an upper bound, it's tight.

A subtlety worth sitting with: an algorithm's *worst-case* behavior and its *average-case* behavior can have different tight bounds. Quicksort's worst case is Θ(n²) (a pathological pivot choice on already-sorted input), but its average case over random inputs is Θ(n log n) — which is why, despite that worst case, it's usually faster in practice than algorithms with a better worst-case guarantee.

## Little-o and little-omega: strict versions

Two notations you'll see less often, little-o (o(f(n))) and little-omega (ω(f(n))), are the *strict* versions of Big-O and Big-Omega: o(f(n)) means g(n) grows strictly slower than f(n) — the ratio g(n)/f(n) goes to zero, not just stays bounded. They're used when you need to say an algorithm is not just "no worse than" but "asymptotically negligible compared to" something else — for instance, stating that an O(n) preprocessing step is o(n log n) relative to an O(n log n) main algorithm, meaning it won't dominate the total cost as n grows.

## Reading complexity off real code

The mechanical version of this skill: count the operations as a function of n. A single loop over n elements is O(n). A loop nested inside another loop, each running n times, is O(n²) — not because there are "two loops" but because the total number of inner-loop executions is n × n. A loop that halves its range each iteration (like binary search, or a recursive function splitting its input in half) contributes a factor of log n. Recognizing these shapes — single pass, nested passes, halving — covers the large majority of complexity analysis you'll do before specialized techniques like recurrence relations become necessary for genuinely recursive algorithms.
