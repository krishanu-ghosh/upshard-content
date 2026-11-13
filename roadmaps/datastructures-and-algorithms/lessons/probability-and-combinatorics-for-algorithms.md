---
title: "Probability and Combinatorics for Algorithms"
description: How expected value, linearity of expectation, and counting arguments explain why randomized quicksort is fast on average, no matter the input.
roadmap: datastructures-and-algorithms
node: probability-and-combinatorics-for-algorithms
group: complexity-and-math-foundations
video:
  provider: youtube
  id: "cNB2lADK3_s"
  start: 0
  end: null
creator:
  name: MIT OpenCourseWare
  channel_url: https://www.youtube.com/@mitocw
duration_seconds: 4800
level: intermediate
prerequisites:
  - asymptotic-notation-big-o-theta-and-omega
takeaways:
  - Big-O analysis alone tells you the worst case; probability lets you reason about the average case, which is what most inputs actually look like.
  - Expected value is a weighted average over outcomes, and linearity of expectation lets you sum the expected contribution of many small events even when those events depend on each other.
  - Randomized quicksort picks its pivot uniformly at random instead of always using, say, the first element, which turns a worst-case-triggering input into just another input with no special power over the algorithm.
  - The classic argument bounds the expected number of comparisons by summing, over every pair of elements, the probability that pivot choices ever compare them directly — that sum collapses into a harmonic series and lands at O(n log n).
  - Basic combinatorics — counting how many ways a set of events can occur — is the tool that turns "how many pairs are there" and "how many orderings are possible" into the numbers a probability argument needs.
check_questions:
  - q: Why does "worst-case O(n²)" not disqualify quicksort as a fast algorithm in practice?
    a: Because the worst case requires a specific adversarial relationship between the pivot-selection rule and the input order, and randomized pivot selection makes that relationship impossible to arrange in advance — every input's expected running time is O(n log n), not just most inputs.
  - q: What does linearity of expectation let you do that summing probabilities directly does not?
    a: It lets you compute the expected value of a sum of random variables as the sum of their individual expected values, even when those variables are not independent — which is exactly the situation with the indicator variables for "were elements i and j ever compared," since one comparison changes the probabilities of later ones.
  - q: In the randomized-quicksort argument, what does X_ij represent, and why is it enough to compute E[X_ij] for every pair?
    a: X_ij is an indicator variable equal to 1 if elements i and j are ever compared during the sort and 0 otherwise; since the total number of comparisons is the sum of X_ij over all pairs, and expectation distributes over that sum by linearity, computing each pairwise E[X_ij] and adding them up gives the expected total comparison count directly.
deeper:
  - label: "Probabilistic Analysis Deep Dive #2: Average Case Complexity, Expected Value & Randomized Quicksort"
    url: "https://www.youtube.com/watch?v=M0O-s2y0O9I"
related_formulas:
  - expected-value
  - conditional-probability
---

## Worst case versus average case

Asymptotic notation, on its own, usually describes the worst case: the single input, out of all possible inputs of size n, that makes an algorithm do the most work. That number matters, but it can also mislead. Quicksort's worst case is O(n²) — it happens when every pivot you pick is the smallest or largest remaining element, so each partition step only shaves one element off the problem. Yet quicksort is one of the fastest sorts in practice, fast enough that it's the default behind many standard-library sort functions. The gap between "worst case is quadratic" and "runs fast in practice" is exactly what probability is for: it lets you ask not "what's the worst that can happen" but "what happens on average, across the inputs and random choices the algorithm actually sees."

## Expected value and linearity of expectation

The tool for "on average" is expected value: the sum, over every possible outcome, of that outcome's value weighted by its probability. If a random variable X takes value 5 with probability 1/2 and value 1 with probability 1/2, its expected value E[X] is 5(1/2) + 1(1/2) = 3 — not a value X can ever actually take, but the long-run average if you repeated the experiment many times.

The property that makes expected value useful for algorithm analysis is linearity of expectation: for any random variables X and Y, E[X + Y] = E[X] + E[Y], full stop — no requirement that X and Y be independent. This is a bigger deal than it looks. Probabilities of combined events are a pain to compute when events interact (the probability that two dependent things both happen isn't just the product of their individual probabilities), but expectations of sums are always just a sum of expectations, dependence or not. That lets you break a complicated random quantity — like "how many comparisons did this sort perform" — into many simple pieces, find the expected value of each piece on its own, and add them up without ever reasoning about how the pieces interact.

## Randomized quicksort: removing the adversary

Ordinary quicksort picks a pivot by some fixed rule — often "the first element" or "the last element" — and partitions the array around it, recursing on each side. The O(n²) worst case happens when the input is already sorted (or reverse sorted) and the fixed rule keeps picking the smallest or largest element as pivot every time. That worst case isn't rare in practice, either — "already sorted" or "nearly sorted" input shows up constantly in real data, which makes a fixed pivot rule a genuine liability, not just a theoretical curiosity.

Randomized quicksort fixes this by picking the pivot uniformly at random from the current subarray on every call, instead of using a fixed position. This doesn't change what the worst case *could* be — if you happened to randomly pick the smallest element every single time, you'd still get O(n²) — but it breaks the connection between "an input is bad" and "the algorithm behaves badly on it." No input can force bad pivot choices anymore, because the choices don't depend on the input at all. Every input, including the previously-worst-case sorted array, now has the *same* expected running time, because that expectation is taken over the algorithm's coin flips, not over some imagined distribution of inputs.

## The worked argument: why it's O(n log n) in expectation

Here's the classic way to make "fast on average" precise. Label the elements by their rank in sorted order, 1 through n, and define an indicator variable X_ij for each pair i < j: X_ij = 1 if elements i and j are ever directly compared to each other during the sort, and 0 if they never are. The total number of comparisons the algorithm performs is exactly the sum of X_ij over all pairs i < j — every comparison the algorithm does involves some specific pair, and each pair either gets compared (contributing 1) or doesn't (contributing 0).

By linearity of expectation, E[total comparisons] = the sum over all pairs of E[X_ij]. And E[X_ij] for a 0/1 variable is just the probability that X_ij equals 1 — the probability that i and j ever get compared. Here's the key combinatorial fact: elements i and j are compared if and only if, among the elements with ranks between i and j inclusive, one of the two endpoints (i or j) is the *first* one chosen as a pivot. If some element strictly between them is chosen as pivot first, i and j end up in different partitions and are never compared to each other again. Since pivots are chosen uniformly at random, and there are (j - i + 1) elements in that range, each one is equally likely to be the first pivot chosen among them — so the probability that it's specifically i or j is 2/(j - i + 1).

Summing 2/(j - i + 1) over all pairs i < j gives a sum that's bounded by 2 times the harmonic series (1 + 1/2 + 1/3 + ... + 1/n), which is the classic result that grows as O(log n). Multiply that by the n pairs sharing roughly that range and the total comes out to O(n log n) — the same bound as the best-case and average deterministic analysis, except now it holds for *every* input, because the randomness lives in the algorithm, not in some assumption about what inputs look like.

## Combinatorics as the counting layer underneath

None of this works without being able to count things correctly: how many pairs (i, j) exist among n elements (n(n-1)/2, a combination), how many orderings a set of pivots could appear in, how many ways a sequence of random choices could play out. Combinatorics — permutations when order matters, combinations when it doesn't — is the layer of counting that probability arguments are built on. When you see an algorithm-analysis argument that starts with "there are C(n, 2) pairs" or "there are n! possible orderings," that's combinatorics supplying the denominator and numerator a probability calculation needs. Together, probability and combinatorics turn "this algorithm feels fast in practice" into a specific, provable expected-case bound — the same move that justifies using randomization, hashing with random seeds, and skip lists throughout the rest of this roadmap.
