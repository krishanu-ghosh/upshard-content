---
title: "Fast Exponentiation"
description: Compute a^b mod m in O(log b) time with repeated squaring instead of O(b) naive multiplication, worked through a numeric example.
roadmap: datastructures-and-algorithms
node: fast-exponentiation
group: number-theory-for-algorithms
video:
  provider: youtube
  id: "D320QeHS0XQ"
  start: 0
  end: null
creator:
  name: codestorywithMIK
  channel_url: https://www.youtube.com/@codestorywithMIK
duration_seconds: 774
level: intermediate
prerequisites: ["gcd-and-modular-arithmetic"]
takeaways:
  - "Naive exponentiation computes a^b with b-1 multiplications, which is O(b) — far too slow once b reaches into the millions or billions, as it does in cryptography and competitive programming."
  - "Fast exponentiation (binary exponentiation / repeated squaring) rewrites the exponent in binary and builds a^b from repeated squaring of a, using only the bits of b that are set to 1."
  - "Squaring a value and halving the exponent each step means the exponent shrinks exponentially, giving O(log b) multiplications instead of O(b)."
  - "Reducing modulo m after every multiplication (not just at the end) keeps intermediate values small, which is what makes a^b mod m practical even when a and b are huge."
  - "The recursive form falls out of one identity: a^b = (a^(b/2))^2 when b is even, and a^b = a * a^(b-1) when b is odd, which directly mirrors the binary digits of b."
check_questions:
  - q: "Why does fast exponentiation only need O(log b) multiplications instead of O(b)?"
    a: "Each step either squares the current result (halving the remaining exponent) or multiplies in one extra factor of a, and both operations cut the amount of 'exponent left to apply' by roughly half per squaring step. Since the exponent is halved repeatedly rather than decremented by 1, the number of steps needed is proportional to the number of bits in b, i.e. log2(b), not to b itself."
  - q: "Why do we reduce modulo m after every single multiplication rather than computing a^b fully and reducing once at the end?"
    a: "a^b for even modest a and b can be astronomically larger than what fits in standard integer types — computing it in full would overflow or require arbitrary-precision arithmetic, defeating the point of a fast algorithm. Reducing mod m after each multiplication keeps every intermediate value bounded below m^2, which fits comfortably in normal integer arithmetic, while mathematically giving the exact same final answer because (x*y) mod m = ((x mod m)*(y mod m)) mod m."
  - q: "How does the binary representation of the exponent b show up in the fast exponentiation algorithm?"
    a: "At each step the algorithm checks whether the current exponent is even or odd — equivalently, whether the lowest bit of b is 0 or 1. If it's 1 (odd), the current power of a is multiplied into the running result; then the base is squared and the exponent is halved (right-shifted by one bit), moving to the next bit. So the algorithm is effectively walking through b's binary digits from least to most significant, squaring the base for every bit position and multiplying it in wherever that bit is a 1."
deeper:
  - label: "cp-algorithms: Binary Exponentiation"
    url: "https://cp-algorithms.com/algebra/binary-exp.html"
related_formulas: []
---

## Why the naive approach falls apart

Computing `a^b` by multiplying `a` by itself `b - 1` times is the obvious approach, and it works fine when `b` is small. But in modular arithmetic contexts — RSA-style cryptography, hashing schemes, combinatorics modulo a prime — exponents routinely reach into the billions. Multiplying a million times to get one answer, and needing to do that for many different exponents, makes the naive O(b) approach unusable. Fast exponentiation (also called binary exponentiation or repeated squaring) gets the same answer using only O(log b) multiplications — for `b` around a billion, that's roughly 30 multiplications instead of a billion.

## The core trick: squaring instead of counting

The method rests on two simple identities, depending on whether the exponent is even or odd:

- If `b` is even: `a^b = (a^(b/2))^2` — compute the smaller power once, then square it.
- If `b` is odd: `a^b = a * a^(b-1)`, and `b - 1` is now even, so the even-case rule applies next.

Each step either halves the exponent (even case) or reduces it by one and makes it even (odd case), and halving dominates — the exponent shrinks exponentially fast rather than linearly. That exponential shrinkage is exactly what turns O(b) work into O(log b) work: the number of steps is bounded by the number of bits in `b`, i.e. `log2(b)`, because each squaring step is really consuming one bit of the exponent's binary representation.

Seen from that angle, the algorithm is just walking through the exponent's binary digits. Say `b` in binary is `b_k b_(k-1) ... b_1 b_0`. The algorithm repeatedly squares a running "power of `a`" value — after `i` steps it holds `a^(2^i)` — and multiplies that value into the answer whenever the corresponding bit `b_i` is 1. The bits that are 0 get skipped (squared through, but not multiplied in); the bits that are 1 each contribute exactly one factor into the final product. This is precisely how `b` gets expressed as a sum of powers of two, and `a^b` as the product of `a` raised to each of those powers.

## Worked example: 3^13

`13` in binary is `1101` (that is, `8 + 4 + 1`). So `3^13 = 3^8 * 3^4 * 3^1`.

Compute the repeated squares of 3, one bit at a time:

- `3^1 = 3`
- `3^2 = 3^1 squared = 9`
- `3^4 = 3^2 squared = 81`
- `3^8 = 3^4 squared = 6561`

Now read off the bits of 13 (`1101`, from most significant to least: 1, 1, 0, 1) and multiply in the powers of 3 that correspond to the 1-bits (positions 3, 2, and 0 — i.e. `3^8`, `3^4`, and `3^1`):

`3^13 = 3^8 * 3^4 * 3^1 = 6561 * 81 * 3 = 1,594,323`

Only 4 squarings and 2 extra multiplications were needed — 6 total multiplications to compute `3^13`, versus 12 multiplications for the naive approach, and the gap widens enormously as the exponent grows: `3^(2^30)` would take 30 squarings here versus over a billion naive multiplications.

## Adding the modulus

For `a^b mod m`, the exact same squaring structure applies, with one addition: every multiplication is immediately followed by `mod m`. So instead of tracking `3^8 = 6561`, you'd track `3^8 mod m` directly — squaring `3^4 mod m` and then reducing again. This keeps every number the algorithm ever handles below `m^2`, regardless of how large the final unreduced answer would have been, which is what makes `a^b mod m` tractable even when `a` and `b` individually have hundreds of digits. The recursive implementation is just a few lines: if `b == 0`, return 1; if `b` is even, recursively compute `half = fastPow(a, b/2, m)` and return `(half * half) % m`; if `b` is odd, return `(a * fastPow(a, b-1, m)) % m`. Each recursive call roughly halves `b`, giving the same O(log b) multiplications, each one kept cheap by the modular reduction.
