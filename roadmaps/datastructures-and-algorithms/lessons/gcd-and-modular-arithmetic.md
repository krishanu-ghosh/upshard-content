---
title: "GCD and Modular Arithmetic"
description: The Euclidean algorithm finds GCD in O(log) steps via gcd(a,b) = gcd(b, a mod b), the foundation for modular arithmetic and inverses.
roadmap: datastructures-and-algorithms
node: gcd-and-modular-arithmetic
group: number-theory-for-algorithms
video:
  provider: youtube
  id: "svBx8u5bMEg"
  start: 0
  end: null
creator:
  name: Neso Academy
  channel_url: https://www.youtube.com/@nesoacademy
duration_seconds: 600
level: intermediate
prerequisites: []
takeaways:
  - "The Euclidean algorithm rests on one identity: gcd(a, b) = gcd(b, a mod b), which lets you replace a pair of numbers with a strictly smaller pair that has the same GCD."
  - "Repeating that replacement shrinks the numbers so fast — at least by a factor related to the golden ratio every two steps — that the algorithm finishes in O(log(min(a, b))) steps, even for huge numbers."
  - "Naive GCD by checking every common divisor up to min(a, b) takes O(min(a, b)) time; the Euclidean algorithm reduces that to logarithmic time without ever factoring either number."
  - "Modular arithmetic treats numbers as wrapping around after reaching a fixed modulus m, so addition and multiplication stay well-defined: (a + b) mod m and (a * b) mod m can be computed by reducing mod m at every step, keeping numbers small."
  - "A modular inverse of a is a number x such that (a * x) mod m = 1 — the modular equivalent of division — and it exists exactly when gcd(a, m) = 1, which is why GCD and modular arithmetic are taught together."
check_questions:
  - q: "Why does gcd(a, b) equal gcd(b, a mod b), and why doesn't replacing a with a mod b change the answer?"
    a: "Any common divisor of a and b must also divide a mod b, because a mod b is just a minus some multiple of b (a - q*b for the quotient q), and a number that divides both a and b divides any integer combination of them. Conversely, any common divisor of b and a mod b also divides a (since a = q*b + (a mod b)). So the set of common divisors of (a, b) is exactly the set of common divisors of (b, a mod b) — meaning their greatest common divisors must be equal too."
  - q: "Why is the Euclidean algorithm considered fast even for very large numbers, say 100-digit numbers?"
    a: "Each step replaces (a, b) with (b, a mod b), and it can be shown the smaller number shrinks by at least roughly a factor of the golden ratio every two steps in the worst case (the Fibonacci-pair case). That means the number of steps grows only with the number of digits, not the size of the numbers themselves — O(log(min(a,b))) steps total, so even enormous numbers finish in a small, predictable number of iterations."
  - q: "Why do we need gcd(a, m) = 1 for a to have a modular inverse mod m, rather than it always existing?"
    a: "A modular inverse x solves a*x ≡ 1 (mod m), which is equivalent to finding integers x, y with a*x + m*y = 1. By the identity underlying the extended Euclidean algorithm, a*x + m*y can only ever produce multiples of gcd(a, m) — so it can equal 1 only when gcd(a, m) = 1. If a and m share any common factor greater than 1, no combination of a and m can ever sum to exactly 1, so no inverse exists."
deeper:
  - label: "cp-algorithms: Euclidean algorithm for computing GCD"
    url: "https://cp-algorithms.com/algebra/euclid-algorithm.html"
related_formulas: []
---

## The slow way, and the insight that fixes it

The greatest common divisor (GCD) of two integers `a` and `b` is the largest number that divides both evenly. The obvious way to compute it — check every number from `min(a, b)` down to 1 and stop at the first one that divides both — works, but it's O(min(a, b)), which is painfully slow once the numbers get into the millions or billions.

The Euclidean algorithm replaces that linear search with a single recursive identity: `gcd(a, b) = gcd(b, a mod b)`. Why is this true? Any divisor shared by `a` and `b` is also a divisor of `a mod b`, because `a mod b` is literally `a - q*b` for some integer quotient `q`, and a number that divides both `a` and `b` divides any combination `a - q*b` of them. The reverse holds too: any divisor shared by `b` and `a mod b` must also divide `a`, since `a = q*b + (a mod b)`. Because the pairs `(a, b)` and `(b, a mod b)` have exactly the same set of common divisors, they necessarily have the same greatest one. The algorithm stops when the second number hits 0 — at that point the first number is the answer, since `gcd(x, 0) = x`.

## Worked example

Compute `gcd(252, 105)`:

- `gcd(252, 105)` → `252 mod 105 = 42` → `gcd(105, 42)`
- `gcd(105, 42)` → `105 mod 42 = 21` → `gcd(42, 21)`
- `gcd(42, 21)` → `42 mod 21 = 0` → `gcd(21, 0)`
- `gcd(21, 0) = 21`

Four steps, no factoring, no trial division — just repeated remainders. Compare that to checking every number from 1 to 105 for a common factor; the Euclidean algorithm reaches the same answer in a handful of steps regardless of how large the inputs are.

## Why it's so fast

Each step replaces `(a, b)` with `(b, a mod b)`, and the second number in the new pair is always strictly smaller than the second number of the old pair (since a remainder is always less than the divisor). It can be proven that in the worst case — when `a` and `b` are consecutive Fibonacci numbers, the pair that shrinks slowest — the smaller value still drops by a factor related to the golden ratio roughly every two steps. That means the number of steps needed grows only with the number of *digits* in the inputs, not their magnitude: the algorithm runs in O(log(min(a, b))) steps. A pair of 100-digit numbers takes only a few hundred steps, not a search through numbers of astronomical size. This logarithmic behavior — buying huge speed by working with remainders instead of the numbers themselves — is why the Euclidean algorithm, invented over two thousand years ago, still underlies modern cryptographic and competitive-programming code.

## Modular arithmetic: working in a finite wraparound world

Modular arithmetic fixes a modulus `m` and treats any two numbers that differ by a multiple of `m` as equivalent — everything "wraps around" after reaching `m`, the way a 12-hour clock wraps after 12. This matters computationally because it keeps numbers small: instead of computing a huge product or sum and reducing it mod `m` at the very end, you can reduce after every operation. `(a + b) mod m` equals `((a mod m) + (b mod m)) mod m`, and the same holds for multiplication: `(a * b) mod m` equals `((a mod m) * (b mod m)) mod m`. This lets algorithms that would otherwise produce astronomically large intermediate numbers (like computing `2^1000`) stay within normal integer ranges by reducing at every step — which is exactly what makes fast exponentiation practical.

Subtraction and addition translate directly into modular arithmetic, but division doesn't — there's no modular "divide by a" operation in the ordinary sense, because remainders don't behave like fractions. Instead, you multiply by a **modular inverse**: a number `x` such that `(a * x) mod m = 1`, playing the role of `1/a`. Finding `x` turns out to depend directly on GCD: a modular inverse of `a` mod `m` exists precisely when `gcd(a, m) = 1` (that is, `a` and `m` share no common factor), and an extended version of the Euclidean algorithm can compute it directly. This is why GCD isn't just a standalone curiosity — it's the gatekeeper that determines whether modular division is even possible, and the mechanism used to compute it when it is. Together, the Euclidean algorithm and modular arithmetic form the base layer that number-theoretic algorithms — fast exponentiation, cryptographic key exchange, combinatorics modulo a prime — are built on top of.
