---
title: "Sieve of Eratosthenes and Primality Testing"
description: Find every prime up to n in O(n log log n) with the sieve, and test a single number for primality in O(sqrt n) with trial division.
roadmap: datastructures-and-algorithms
node: sieve-of-eratosthenes-and-primality-testing
group: number-theory-for-algorithms
video:
  provider: youtube
  id: "g5Fuxn_AvSk"
  start: 0
  end: null
creator:
  name: take U forward
  channel_url: https://www.youtube.com/@takeUforward
duration_seconds: 1106
level: intermediate
prerequisites: []
takeaways:
  - "Trial division tests whether a single number n is prime by checking divisibility only up to sqrt(n), because any factor pair (d, n/d) must have one member at or below the square root."
  - "The Sieve of Eratosthenes finds every prime up to n by crossing out multiples, starting each prime's crossing-out at p*p rather than 2p, since smaller multiples were already crossed out by smaller primes."
  - "The sieve's total work sums to a harmonic-like series over the primes, which converges to O(n log log n) — almost linear, and far faster than running trial division n separate times."
  - "A sieve amortizes: the O(n log log n) cost is paid once and then every primality question up to n is answered in O(1) by checking a boolean array, which is the right tradeoff whenever you need many answers."
  - "Trial division is the better tool when you only need to test one specific number, especially a large one far bigger than any n you'd want to allocate a sieve array for."
check_questions:
  - q: "Why does trial division only need to check divisors up to sqrt(n), instead of up to n/2?"
    a: "If n has a factor d greater than sqrt(n), then n/d must be a factor smaller than sqrt(n) (since d * (n/d) = n and both can't be above sqrt(n)). So any composite number is guaranteed to have a factor at or below its square root; if none exists there, none exists above it either, and n must be prime."
  - q: "In the sieve, why does marking multiples of a prime p start at p*p instead of 2p?"
    a: "Every multiple of p smaller than p*p is of the form p*k for some k < p, and k itself has a prime factor smaller than p, so that multiple was already crossed out when an earlier, smaller prime was processed. Starting at p*p skips redundant work without missing any composite."
  - q: "You need to answer 'is n prime?' for 100,000 different numbers, all under 10 million. Why build a sieve instead of running trial division 100,000 times?"
    a: "A single sieve over 10 million costs about O(n log log n), roughly 10 million operations total, and after that every query is an O(1) array lookup. Trial division would cost up to O(sqrt(n)) per query, around 3,000 operations each, times 100,000 queries — hundreds of millions of operations. The sieve's one-time cost amortizes across all the repeated questions."
deeper:
  - label: "cp-algorithms: Sieve of Eratosthenes"
    url: "https://cp-algorithms.com/algebra/sieve-of-eratosthenes.html"
related_formulas: []
---

## Testing one number: trial division

The most direct way to check whether a number `n` is prime is to try dividing it by every integer from 2 up to `n - 1` and see if anything divides evenly. That works, but it's wasteful. The key observation that speeds it up: if `n` has any factor, it has a factor at or below `sqrt(n)`. Suppose `n = d * k` with `d <= k`. Then `d` can't be bigger than `sqrt(n)`, because if it were, `k` (which is at least as big as `d`) would make `d * k` bigger than `n`. So factors always come in pairs straddling the square root, and you only need to search up to it — check divisibility by 2, 3, 4, ... up to `floor(sqrt(n))`, and if nothing divides evenly, `n` is prime. For `n` around a million, that's checking roughly 1,000 candidates instead of a million — an O(sqrt(n)) algorithm instead of O(n). A small further optimization: after checking 2, you only need to check odd divisors, since any even divisor would have been caught by 2 already.

This is the right tool when you have one number, possibly a large one, and need a yes/no answer. It does not help if you need that answer for thousands of different numbers — you'd be redoing the square-root-sized search from scratch each time.

## Finding every prime up to n: the sieve

The Sieve of Eratosthenes flips the problem around. Instead of asking "is this number prime?" one at a time, it finds every prime up to some limit `n` in a single pass. Start with a boolean array of size `n + 1`, all initialized to "prime" (true), with index 0 and 1 marked false. Then, starting from `p = 2`, if `p` is still marked prime, cross out every multiple of `p` — `2p, 3p, 4p, ...` — as composite. Move to the next number still marked prime, and repeat. When you've processed every `p` up to `sqrt(n)`, everything still marked true is prime.

Two details make this fast rather than merely correct. First, you only need to sieve with primes up to `sqrt(n)`, not all the way to `n` — any composite number `n` or smaller has at least one prime factor at or below its own square root, so by the time `p` exceeds `sqrt(n)`, every remaining composite has already been crossed out by a smaller prime. Second, when you start crossing out multiples of `p`, you can start at `p*p` instead of `2p`. Every smaller multiple of `p` — `2p, 3p, ..., (p-1)*p` — has a factor smaller than `p` (namely whichever of 2, 3, ..., p-1 it's a multiple of), and that smaller factor already crossed it out in an earlier pass. Starting at `p*p` skips that redundant work entirely.

## Worked example: sieving up to 30

Array indices 0..30, all true except 0 and 1. Start at `p = 2`: cross out 4, 6, 8, ..., 30 (starting at 2*2 = 4). Next surviving number is `p = 3`: cross out 9, 12, 15, ..., 30 (starting at 3*3 = 9; 6 was already crossed out by 2). Next is `p = 5`: start at `5*5 = 25`, cross out 25 and 30. Next would be `p = 7`, but `7*7 = 49 > 30`, so we stop — anything left uncrossed is prime. What survives: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29. That's every prime up to 30, found without a single division — only additions (to compute the next multiple) and array writes.

## Why the sieve amortizes so well

The total work the sieve does isn't `n` operations per prime; it's `n/p` operations for each prime `p` (the number of multiples of `p` up to `n`). Summing `n/2 + n/3 + n/5 + n/7 + ...` over all primes up to `n` gives a sum that grows like `n * log(log(n))` — a function that grows unimaginably slowly; `log(log(n))` is under 4 even for `n` in the billions. So building the whole sieve costs O(n log log n), only a constant factor worse than simply reading every number from 1 to n once.

That cost is paid exactly once. After it, the boolean array answers "is k prime?" for any `k <= n` in O(1) — a single array lookup. This is the amortization payoff: if you need one primality answer, trial division's O(sqrt(n)) per query is cheaper than building an entire sieve. But the moment you need many answers — primes in a range, factorizing many numbers, answering repeated primality queries in a competitive programming problem — the sieve's fixed upfront cost gets divided across every subsequent query, and it wins decisively. Choosing between them is really a choice about how many questions you're going to ask.
