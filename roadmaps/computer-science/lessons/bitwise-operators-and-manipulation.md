---
title: "Bitwise Operators & Bit Manipulation"
description: AND, OR, XOR, shifts, and the bit tricks built on them — checking, setting, clearing, and toggling individual bits.
roadmap: computer-science
node: bitwise-operators-and-manipulation
group: representation-and-caching
video:
  provider: youtube
  id: "SSvHrrbhCCI"
  start: 0
  end: null
creator:
  name: Fullstack Academy
  channel_url: https://www.youtube.com/@FullstackAcademy
duration_seconds: 600
level: beginner
prerequisites: []
takeaways:
  - The bitwise operators AND, OR, XOR, and NOT work on the binary representation of a number directly, one bit position at a time, independent of what the number "means" in decimal.
  - AND with a mask of 1s and 0s reads out specific bits, OR sets specific bits to 1, and XOR toggles specific bits — flipping a 1 to 0 or a 0 to 1 wherever the mask has a 1.
  - Left-shifting a number by k positions multiplies it by 2^k, and right-shifting (on an unsigned or non-negative value) divides it by 2^k, both far faster than general multiplication or division.
  - XOR's self-cancelling property (a value XORed with itself is 0, and XORed with 0 is unchanged) is the basis for swapping two variables without a temporary one and for finding the single number that appears an odd number of times in a list.
  - Counting the number of set bits in an integer, or checking whether it's a power of two, reduces to a handful of bitwise operations that run in constant or near-constant time rather than looping over its decimal digits.
check_questions:
  - q: How do you check whether the 3rd bit (counting from 0) of an integer `n` is set?
    a: Compute `n & (1 << 3)`; if the result is nonzero, that bit is 1, because the mask has a single 1 in exactly that position and AND only keeps bits where both operands have a 1.
  - q: Why does `x ^ x` always equal 0, and why does that make XOR useful for finding a "lonely" number in a list where every other number appears twice?
    a: XOR compares corresponding bits and outputs 1 only when they differ, so identical values always cancel to 0; XORing every number in the list together cancels every pair that appears twice, leaving only the number that appeared once.
  - q: What's a quick way to check if a positive integer n is a power of two using bitwise operators?
    a: Check whether `n & (n - 1) == 0`, because a power of two has exactly one bit set, and subtracting 1 flips that bit to 0 and every bit below it to 1, so ANDing the two leaves nothing in common.
deeper:
  - label: "Unlocking Bit Manipulation Secrets!"
    url: "https://www.youtube.com/watch?v=5kvgF0xN6SM"
related_formulas: []
---

## Operating below the number

Every integer a program works with is, underneath, a sequence of bits — 0s and 1s. Bitwise operators let you manipulate that sequence directly, position by position, rather than treating the number as an indivisible decimal quantity. They're the fastest operations a CPU performs, which is why they show up in performance-sensitive code, in low-level protocols that pack several fields into one integer, and in a cluster of classic interview problems that have nothing to do with raw speed and everything to do with elegance.

## The four core operators

**AND (`&`)** compares two numbers bit by bit and outputs 1 only where *both* have a 1. It's used to "mask" a number — keep only the bits you care about and zero out the rest. **OR (`|`)** outputs 1 where *either* operand has a 1, which is how you set specific bits to 1 without disturbing the others. **XOR (`^`, exclusive or)** outputs 1 where the two bits *differ* — this makes it a toggle: XORing a bit with 1 flips it, XORing with 0 leaves it alone. **NOT (`~`)** flips every bit in a single number, 1s to 0s and 0s to 1s.

Picture the number 12 (`1100` in binary) and the mask 10 (`1010`). `12 & 10` = `1000` (8) — only the third bit is 1 in both. `12 | 10` = `1110` (14) — a bit is 1 if either has it. `12 ^ 10` = `0110` (6) — only the bits that differ survive.

## Shifts: multiplication and division without the arithmetic

A left shift (`<<`) moves every bit one position to the left and fills the vacated spot with 0, which is equivalent to multiplying by 2 for each position shifted: `5 << 2` turns `101` into `10100`, which is 20 — 5 × 4. A right shift (`>>`) does the reverse, dividing by 2 per position (for non-negative numbers): `20 >> 2` recovers 5. Because shifting is a single, cheap CPU instruction, code that needs to multiply or divide by a power of two — sizing a buffer, computing an array index into a power-of-two-sized hash table — often does it with a shift instead of a general multiply.

## XOR's party trick: self-cancellation

XOR has one property that makes it unusually useful: `x ^ x = 0` for any `x`, and `x ^ 0 = x`. That means XOR-ing a value with itself erases it, and XOR-ing with 0 is a no-op. Two consequences follow directly. First, you can swap two variables `a` and `b` without a temporary variable: `a ^= b; b ^= a; a ^= b;` — each step unwinds the previous one using the self-cancellation property. Second, if you XOR together every number in a list where every value appears exactly twice except one, every pair cancels to 0 and only the unpaired value survives — a single pass, constant extra memory, no hash set required.

## Bit tricks worth recognizing

A handful of bitwise patterns recur often enough to be worth memorizing outright, because they replace a loop with one or two operations:

- **Check a bit:** `n & (1 << k)` is nonzero exactly when bit `k` of `n` is set.
- **Set a bit:** `n | (1 << k)` turns bit `k` on without touching the others.
- **Clear a bit:** `n & ~(1 << k)` turns bit `k` off.
- **Toggle a bit:** `n ^ (1 << k)` flips bit `k`.
- **Check if n is a power of two:** `n > 0 && (n & (n - 1)) == 0`. A power of two has exactly one bit set; subtracting 1 flips that bit off and every bit below it on, so the two numbers share no 1-bits.
- **Count set bits (Brian Kernighan's trick):** repeatedly do `n = n & (n - 1)`, which clears the lowest set bit each time; the number of iterations until `n` reaches 0 is the count of 1-bits, done in time proportional to the number of set bits rather than the number of total bits.

None of these require understanding anything exotic — they fall directly out of what AND, OR, XOR, and shifts do to a binary representation, which is why they're worth deriving once rather than memorizing as disconnected facts.
