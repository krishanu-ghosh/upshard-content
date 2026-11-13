---
title: "Boyer-Moore voting algorithm"
description: Finding the element that appears more than n/2 times in O(n) time and O(1) space via candidate-and-counter cancellation.
roadmap: datastructures-and-algorithms
node: boyer-moore-voting-algorithm
group: more-patterns-and-tricks
video:
  provider: youtube
  id: "7pnhv842keE"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 878
level: intermediate
prerequisites: [arrays-linked-lists-stacks-and-queues]
takeaways:
  - "The Boyer-Moore voting algorithm finds the majority element (one appearing more than n/2 times) in a single O(n) pass using only O(1) extra space — one candidate variable and one integer counter, no hash map."
  - "The sweep logic: if the counter is 0, adopt the current element as the new candidate; if the current element equals the candidate, increment the counter; otherwise decrement it — a non-majority element effectively 'cancels out' one occurrence of the current candidate."
  - "Correctness relies on a pairing argument: every time the counter is decremented, you can think of it as removing one copy of the candidate and one copy of some other, different element as a canceling pair; because the true majority element occurs more than n/2 times, it cannot be fully canceled out by all the other elements combined, so it necessarily survives as the final candidate."
  - "The algorithm only guarantees a correct answer when a true majority element is known to exist; if the input might not have one, a required second pass counts the actual occurrences of the final candidate to verify it exceeds n/2 before trusting the result."
  - "The algorithm generalizes to finding all elements appearing more than n/k times by tracking k-1 candidate-and-counter pairs simultaneously, since at most k-1 elements can each exceed a 1/k share of the array; the classic extension (n/3) uses exactly 2 candidate-counter pairs, with a decrement happening only when an element matches neither current candidate."
  - "For the n/3 extension, a verification pass is still required at the end, since the two surviving candidates are only guarantees of 'possibly exceeds n/3' — the algorithm can produce candidates that turn out not to actually satisfy the threshold if no such majority exists."
check_questions:
  - q: "Why does the 'cancellation' logic in Boyer-Moore voting guarantee the true majority element survives as the final candidate, when the algorithm never actually counts total occurrences during the sweep?"
    a: "Each decrement effectively pairs off one occurrence of the current candidate with one occurrence of a different element, canceling both out of consideration. Because the true majority element appears more than n/2 times, there aren't enough 'other' elements in the entire array to pair against and cancel every one of its occurrences — at least one occurrence must remain unpaired at the end, which is why it always ends up as (or back as) the surviving candidate, regardless of the order elements appear in."
  - q: "Why is a second verification pass necessary after the main sweep, rather than just trusting whatever candidate comes out?"
    a: "The algorithm's cancellation logic only produces a correct majority element if one is guaranteed to exist in the input; if no element actually occurs more than n/2 times, the sweep will still produce some candidate (since the logic always ends with something in the candidate slot), but that candidate may not actually be a majority element. A second pass that counts the candidate's real occurrences and checks it exceeds n/2 is the only way to confirm the answer is valid rather than a false positive from an input with no majority."
  - q: "Why does finding elements appearing more than n/3 times require tracking two candidates and two counters instead of one?"
    a: "At most two distinct elements can each individually exceed a 1/3 share of an array of size n, because three elements each exceeding n/3 would already sum to more than n total occurrences, which is impossible. Tracking exactly k-1 = 2 candidate-counter pairs for the n/3 case covers every possible majority-by-more-than-a-third element; one pair, as in the n/2 case, isn't enough capacity to represent two simultaneously-possible answers."
deeper:
  - label: "Majority Element II - Leetcode 229 - Python (the n/3 extension) — NeetCodeIO"
    url: "https://www.youtube.com/watch?v=Eua-UrQ_ANo"
related_formulas: []
---

## The problem and the naive approaches

Given an array where one value is guaranteed to occur more than `n/2` times (the
"majority element"), find it. A hash map counting occurrences solves this in O(n) time
but O(n) space. Sorting the array and reading the middle element also works (the
majority element, by definition, must occupy the middle position of a sorted array) but
costs O(n log n). Boyer-Moore voting gets O(n) time **and** O(1) space by never counting
anything explicitly — it tracks a single running "candidate" and lets non-matching
elements cancel it out.

## The sweep, and the cancellation intuition

Maintain two variables: `candidate` (initially unset) and `count = 0`. Walk the array
once:

- If `count == 0`, set `candidate = nums[i]` and `count = 1`.
- Else if `nums[i] == candidate`, increment `count`.
- Else, decrement `count`.

The intuition: think of `count` as a "net vote" for the current candidate. Every element
that disagrees with the candidate casts one vote against it. If the candidate and a
disagreeing element are thought of as a canceling pair — one occurrence of the candidate
destroyed alongside one occurrence of something else — then after processing the whole
array, how many of the majority element's occurrences could possibly have been
destroyed? At most `n - (majority count)` of them, since each cancellation consumes
exactly one non-candidate element to pair against. Since the majority element occurs
more than `n/2` times, there are fewer than `n/2` other elements total — not enough to
cancel out every occurrence of the majority element, no matter how they're arranged.
So at least one occurrence of the true majority element survives the whole sweep,
guaranteeing it ends up as (or returns as) the final candidate.

## Worked example

`nums = [2, 2, 1, 1, 1, 2, 2]` (n=7, majority element is 2, appearing 4 times > 7/2).

- `i=0`: `count=0` → `candidate=2, count=1`.
- `i=1`, `nums[1]=2`: matches → `count=2`.
- `i=2`, `nums[2]=1`: doesn't match → `count=1`.
- `i=3`, `nums[3]=1`: doesn't match → `count=0`.
- `i=4`, `nums[4]=1`: `count=0` → `candidate=1, count=1`.
- `i=5`, `nums[5]=2`: doesn't match → `count=0`.
- `i=6`, `nums[6]=2`: `count=0` → `candidate=2, count=1`.

Final candidate: `2` — correct, even though the candidate changed mid-sweep and `count`
hit zero twice. Each zero-crossing represents a moment where the running tally of votes
for whatever was leading got fully canceled by disagreement, but the true majority
element (2) had enough total occurrences to keep reclaiming the candidate slot and
finish on top.

## Why a verification pass matters

The algorithm as described always outputs *some* candidate, even if no true majority
element exists (e.g., `[1, 2, 3]`). The cancellation argument only proves correctness
*given* that a majority element is guaranteed to exist; without that guarantee, a second
linear pass that counts the candidate's actual occurrences and checks it exceeds `n/2`
is required before trusting the result.

## Generalizing to n/3

The same idea extends to finding every element that appears more than `n/k` times, by
observing that at most `k-1` distinct elements can each exceed a `1/k` share of the
array (if `k` elements each exceeded `n/k`, their occurrences would sum past `n`, which
is impossible). For the common "more than n/3" variant, track **two** candidates and
two counters simultaneously: on each element, try to match or fill an empty candidate
slot first; only decrement both counters together when the element matches neither
current candidate. After the sweep, a verification pass counts actual occurrences of
both surviving candidates, since the sweep alone only narrows down the possibilities —
it doesn't guarantee either surviving candidate is a real answer if no such element
exists.
