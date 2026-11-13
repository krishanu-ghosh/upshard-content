---
title: "Randomized Algorithms: Las Vegas vs Monte Carlo"
description: How randomness turns adversarial worst cases into expected-case guarantees, splitting algorithms into always-correct Las Vegas and bounded-time Monte Carlo.
roadmap: datastructures-and-algorithms
node: randomized-algorithms
group: algorithm-design-techniques
video:
  provider: youtube
  id: "cNB2lADK3_s"
  start: 0
  end: null
creator:
  name: MIT OpenCourseWare
  channel_url: https://www.youtube.com/@mitocw
duration_seconds: 4912
level: intermediate
prerequisites: ["order-statistics-and-quickselect", "amortized-analysis"]
takeaways:
  - "A randomized algorithm makes some of its decisions using a random number generator rather than purely from the input, which means its running time (or, for some algorithms, its correctness) becomes a random variable instead of a fixed function of input size."
  - "Las Vegas algorithms always produce a correct answer; what's random is their running time, which is bounded only in expectation, not in the worst case."
  - "Monte Carlo algorithms always finish in a fixed, bounded running time; what's random is their correctness, which holds only with some probability (often made arbitrarily high by repeating independent trials)."
  - "Randomized quicksort picks its pivot uniformly at random each call instead of always picking, say, the first or last element, which makes the already-sorted/reverse-sorted adversarial input that triggers deterministic quicksort's O(n²) worst case no longer special — every input now has the same expected O(n log n) running time."
  - "Expected running time is an average over the algorithm's own internal coin flips for a fixed input, not an average over possible inputs — the guarantee holds even on the single worst input an adversary could choose, because the adversary can't predict or control the random choices."
  - "Any Las Vegas algorithm can be converted into a Monte Carlo one by capping its running time and returning an arbitrary answer if it doesn't finish in time; converting the other direction generally requires an efficient way to verify a Monte Carlo algorithm's output, so it can be rerun until a checkable-correct answer appears."
check_questions:
  - q: "Deterministic quicksort (always pivot on the first element) has a clean, reproducible O(n²) worst case on already-sorted input. Randomized quicksort (uniform random pivot) can also pick a terrible pivot on any given run. Why do we still say randomized quicksort 'solves' the worst-case problem?"
    a: "The difference is which worst case matters. Deterministic quicksort's worst case is tied to the *input*: sorted input reliably and reproducibly triggers it every single run, so an adversary (or just unlucky real-world data, like nearly-sorted logs) can reliably force bad performance. Randomized quicksort's pivot choice is independent of the input, so no input — not even one an adversary designs specifically to attack it — can reliably force a bad pivot sequence; a bad run is possible but vanishingly unlikely, and its probability doesn't depend on what the input looks like. The O(n²) case still technically exists for randomized quicksort, but it's no longer something an adversary can trigger on purpose."
  - q: "What's the precise difference between 'expected running time' and 'average-case running time', and why does that distinction matter for randomized quicksort's guarantee?"
    a: "Average-case running time averages over a distribution of possible *inputs* (e.g., 'assume all permutations are equally likely') — it says nothing about what happens on a specific adversarial input, which might not be drawn from that nice distribution. Expected running time, for a randomized algorithm, averages over the algorithm's own internal random choices for a single *fixed* input (even one an adversary deliberately constructed). Randomized quicksort's O(n log n) expected bound holds for every fixed input individually, because the randomness being averaged over is the pivot choices the algorithm itself makes, not some assumption about which inputs are likely to occur."
  - q: "A Monte Carlo algorithm for a decision problem is correct with probability at least 1/2 on each run and runs in fixed time T. Why does running it k independent times and taking a majority vote reduce the error probability, and what has to be true for this trick to work at all?"
    a: "Each independent run has an independent chance of being wrong, so the probability that a majority of k runs are simultaneously wrong shrinks exponentially in k (by a Chernoff-type bound), even though no single run's guarantee improved. This only works because the runs are genuinely independent — each uses fresh randomness uncorrelated with the others — and because the error is one-sided or at least consistent enough that 'majority of correct answers' is a meaningful signal; if the algorithm's correctness probability were tied to some hidden property of the input that didn't change between runs, repeating it wouldn't actually reduce the error at all."
deeper:
  - label: "MIT 6.046J Lecture 6: Randomization — Matrix Multiply, Quicksort (course page with notes)"
    url: "https://ocw.mit.edu/courses/6-046j-design-and-analysis-of-algorithms-spring-2015/resources/lecture-6-randomization-matrix-multiply-quicksort/"
related_formulas: []
---

## Letting the algorithm flip its own coins

A randomized algorithm has access to a source of random bits and uses them to make some of its internal decisions — which element to pick as a pivot, which hash function to use, which direction to branch — rather than deriving every decision deterministically from the input. This sounds like a small change, but it fundamentally shifts what "running time" and "correctness" mean: instead of being fixed functions of the input, they become random variables, because the same input can produce different execution paths on different runs.

Once running time or correctness is a random variable, there are two fundamentally different kinds of guarantee you can make about it, and the two families of randomized algorithms are named after them.

## Las Vegas: randomness in the runtime, never in the answer

A **Las Vegas algorithm** always produces the correct answer. What varies from run to run is how long it takes. The guarantee you prove is a bound on *expected* running time — the average over the algorithm's internal random choices, for a worst-case or even adversarially chosen fixed input.

**Randomized quicksort** is the canonical example. Standard quicksort partitions around a pivot and recurses on both sides; its performance hinges entirely on how balanced the partition is. If you always pick a fixed position as the pivot (say, the first element), an adversary — or simply unlucky, structured input like already-sorted data — can force every partition to be maximally unbalanced (one side empty, one side n-1 elements), producing the familiar O(n²) worst case, and that input is reproducible and discoverable.

Randomized quicksort picks the pivot **uniformly at random** from the current subarray on every call. Now, for any fixed input, the sequence of partition qualities is itself a random variable, and the question becomes: what's the expected number of comparisons? The key insight is that two elements are compared only if one of them is chosen as a pivot before any element strictly between them in sorted order — this happens with probability inversely related to their rank distance, and summing that probability over all pairs gives an expected total of O(n log n) comparisons, *for every input*, including the one that was worst-case for the deterministic version. The algorithm still outputs a correctly sorted array every single time (it's sorting — correctness isn't in question); only the time to get there is a random variable, and its expectation is good no matter what the input looks like.

## Monte Carlo: randomness in the answer, never in the runtime

A **Monte Carlo algorithm** runs in a fixed, bounded amount of time on every input and every run — but it might return a wrong answer, with some bounded probability. The classic example is checking whether a claimed matrix product C = A·B is correct: computing A·B directly to verify takes as long as the multiplication itself, but picking a random vector r and checking whether A(Br) = Cr can be done in the time of a few matrix-vector products. If C ≠ AB, the check only misses the error for a specific, bounded slice of "unlucky" vectors r — for the standard construction, that slice is at most half of them, which is exactly why a single run isn't enough confidence on its own and the trick from the next section (run it several times) is what drives the error down further. The algorithm finishes fast every time; what's uncertain is whether its "yes, correct" answer is actually right.

## Boosting confidence and converting between the two

A Monte Carlo algorithm's error probability can usually be driven down by independent repetition — run it k times and take a majority vote (or, for one-sided error, accept any single "found a witness" result), and the chance all k trials are simultaneously wrong shrinks exponentially in k, at the cost of a k-times slowdown.

The two families are related but not symmetric. Any Las Vegas algorithm converts easily into a Monte Carlo one: just cap its running time at some threshold and return a default or arbitrary answer if it hasn't finished — you trade the "always correct" guarantee for a hard time bound. Going the other direction — turning a Monte Carlo algorithm into a Las Vegas one — generally requires an efficient way to *verify* whether a given output is correct, so you can rerun the algorithm until it produces a verifiably-correct answer. That isn't always possible. Not every problem admits a cheap correctness check, which is exactly why Monte Carlo algorithms are often used in settings where no Las Vegas equivalent is known to exist.
