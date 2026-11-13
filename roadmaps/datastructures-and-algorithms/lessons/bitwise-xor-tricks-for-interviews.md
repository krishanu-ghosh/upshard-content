---
title: "Bitwise XOR Tricks for Interviews"
description: The XOR self-cancellation trick behind single-number problems, plus using a small bitmask as a stand-in for small-set membership.
roadmap: datastructures-and-algorithms
node: bitwise-xor-tricks-for-interviews
group: more-interview-patterns
video:
  provider: youtube
  id: "qMPX1AOa83k"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 428
level: intermediate
prerequisites: []
takeaways:
  - "XOR has two properties that make it an interview shortcut: a^a = 0 (anything XORed with itself cancels to zero) and a^0 = a (anything XORed with zero is unchanged)."
  - "XORing every element of an array where every value appears exactly twice except one leaves only the unpaired value, because the duplicated values cancel each other out in pairs and 0 XORed with the survivor leaves it untouched — done in O(n) time and O(1) space, beating a hash-set approach on space."
  - "XOR is both commutative and associative, which is the quiet fact that makes the single-number trick work regardless of the array's order — you can XOR the elements in any sequence and get the same result."
  - "The 'missing number from 1..n' variant XORs the indices 0..n together with the array's values together — every present number cancels with its matching index, leaving only the missing one, again without needing a sum formula or a visited array."
  - "A small bitmask (an integer where bit i means 'element i is in the set') represents set membership in a single machine word — adding element i is `mask |= (1 << i)`, checking membership is `mask & (1 << i)`, and this is the standard shortcut for 'set of up to ~20-30 small items' problems where a hash set would be overkill."
check_questions:
  - q: "Why does XORing every element of [4, 1, 2, 1, 2] together correctly return 4, the single non-duplicated value?"
    a: "Because XOR is commutative and associative, the order of operations doesn't matter, so the 1s can be grouped together (1^1 = 0) and the 2s can be grouped together (2^2 = 0), leaving 0^0^4 = 4 — every duplicate pair cancels to zero and only the unmatched value survives."
  - q: "Why does the missing-number trick XOR the array values together with the range 0..n, rather than just XORing the array values alone?"
    a: "XORing the array values alone would only cancel pairs that already exist within the array itself, which doesn't help when every array value is unique and one expected value is simply absent; by also XORing in every expected index 0..n, each present value cancels against its matching index, and the one index with no matching value left in the array is exactly the missing number."
  - q: "Why is a bitmask integer a reasonable substitute for a hash set only when the universe of possible elements is small (roughly ≤ 30-64 items)?"
    a: "Because a bitmask needs one bit per possible element to represent membership, so it only fits in a single machine integer (32 or 64 bits) when the total number of distinct possible elements is small; beyond that it would need a bit-array or multiple integers, at which point the O(1)-word tricks like single-popcount lookups or single-instruction union/intersection stop being a clean win over a hash set."
deeper:
  - label: "Bitwise Hacks for Competitive Programming"
    url: "https://www.geeksforgeeks.org/bitwise-hacks-for-competitive-programming/"
related_formulas: []
---

## Two facts about XOR, and why they're worth memorizing

Everything in this lesson follows from two one-line facts about the XOR operator (`^`): `a ^ a = 0` (a value XORed with itself is zero), and `a ^ 0 = a` (a value XORed with zero is unchanged). Combined with the fact that XOR is both commutative (`a^b = b^a`) and associative (`(a^b)^c = a^(b^c)`), these mean you can XOR a whole collection of numbers together in any order, and any value that appears an even number of times will cancel itself out completely, leaving only whatever appears an odd number of times (or doesn't match up with something else). That single idea is the entire "XOR trick" — it's not really one trick, it's one algebraic fact applied to a few different setups.

## The single number problem, worked

The classic version: given an array where every element appears exactly twice except for one, find the one that appears once. Take `[4, 1, 2, 1, 2]`. Initialize `result = 0` and XOR in every element: `0^4 = 4`, `4^1 = 5`, `5^2 = 7`, `7^1 = 6`, `6^2 = 4`. Final result: `4`. Nothing in that computation ever looked at which positions held duplicates — the pairing happened implicitly, because by the time all five values have been folded in, the two 1s have cancelled each other (in whatever order they were encountered) and the two 2s have cancelled each other, leaving only `0^4`. This runs in a single O(n) pass with O(1) extra space, which is strictly better than the hash-set alternative (track seen values, return the one that was never removed) that uses O(n) space to do the same job.

## The missing number variant

A related but distinct problem: given an array of n distinct numbers taken from the range `0..n` (so one number from that range is missing), find the missing one. XORing just the array elements together doesn't help here, since nothing in the array is duplicated. The trick is to XOR in *both* the array's values and every index from `0` to `n`: every number that's actually present in the array will appear exactly twice across the combined set (once as a value, once as its matching index), and the number that's missing from the array will appear exactly once (only as an index). For `[3, 0, 1]` (expected range `0..3`, so the missing value is 2): XOR `0^1^2^3` (the indices) with `3^0^1` (the values) — `0^1^2^3^3^0^1`. Group and cancel: the two `0`s cancel, the two `1`s cancel, the two `3`s cancel, leaving just `2`. No sum formula (`n(n+1)/2` minus the array sum) is needed, and this version is immune to the integer-overflow edge case that a sum-based approach can hit on very large inputs.

## Bitmask as a stand-in for small-set membership

The second XOR-adjacent trick isn't about finding a missing value. It's about representing an entire *set* of small items as a single integer. If you have at most ~30 possible elements (say, indices 0 through 25 for the lowercase alphabet), a set of "which ones are present" packs into one integer's bits: bit `i` is 1 if element `i` is in the set.

Four operations, four one-liners. Adding element `i` is `mask |= (1 << i)`. Removing it is `mask &= ~(1 << i)`. Checking membership is `mask & (1 << i)` (nonzero means present). And *toggling* element `i` — add it if absent, remove it if present — is `mask ^= (1 << i)`, the same self-cancellation idea from the single-number trick, just applied one bit at a time instead of to a whole array.

This is why problems that look like "track which subset of a small group has been visited" (common in bitmask-DP setups) often carry a single integer through the recursion instead of a hash set or boolean array. Comparing, combining, or copying two sets becomes one machine-word operation instead of an `O(n)` structure traversal.
