---
title: "Linear Search, Binary Search, and Recursion"
description: Why binary search needs sorted data, how recursion replaces a loop with a call stack, and what makes a recursive call "tail" or not.
roadmap: datastructures-and-algorithms
node: searching-algorithms-and-recursion
group: sorting-searching-and-recursion
video:
  provider: youtube
  id: "gl3emqCuueQ"
  start: 0
  end: null
creator:
  name: Stanford
  channel_url: https://www.youtube.com/@stanford
duration_seconds: 2556
level: beginner
prerequisites:
  - comparison-based-sorting-algorithms
takeaways:
  - Linear search checks every element in order and works on any list, sorted or not, in O(n) time, while binary search needs the data sorted first but then finds a target in O(log n) by repeatedly halving the range to search.
  - Binary search works by comparing the target to the middle element and discarding the half of the range that can't contain it, which is only valid because a sorted order guarantees everything on one side is smaller and everything on the other is larger.
  - A recursive function solves a problem by calling itself on a smaller version of the same problem, and every recursive function needs a base case — a version of the problem small enough to answer directly — or it recurses forever.
  - Each recursive call pushes a new frame onto the call stack to hold that call's local variables and return address, so a recursion that goes too deep without reaching its base case causes a stack overflow, the same failure mode as an infinite loop that never terminates.
  - A tail-recursive call is one where the recursive call is the very last thing the function does, with no pending work left to do after it returns; that shape lets some languages and compilers reuse the current stack frame instead of pushing a new one, while a non-tail-recursive call (like one still waiting to multiply its result after the recursive call returns) must keep its frame alive and so cannot get that optimization.
check_questions:
  - q: Why can't binary search be used on an unsorted array, even if you know the target is present?
    a: Binary search decides which half to discard by comparing the target to the middle element and trusting that everything smaller is to one side and everything larger is to the other; without a sorted order that assumption is false, so the half it throws away could still contain the target.
  - q: What happens if a recursive function is missing its base case?
    a: With no condition that stops the recursion, the function keeps calling itself on smaller (or even unchanged) inputs indefinitely, pushing a new stack frame on every call until the call stack runs out of space and the program crashes with a stack overflow.
  - q: Is a recursive factorial function written as `return n * factorial(n - 1)` tail-recursive? Why or why not?
    a: No — after `factorial(n - 1)` returns, the function still has to multiply that result by `n` before it can return, so there's pending work after the recursive call, which is exactly what disqualifies it from being tail-recursive.
deeper:
  - label: "Tail Recursion Explained - Computerphile"
    url: "https://www.youtube.com/watch?v=_JtPhF8MshA"
related_formulas: []
---

## Linear search: the baseline

Linear search is the simplest way to find something: walk the list from the start, compare each element to the target, and stop when you find it (or reach the end without finding it). It makes no assumptions about the data — unsorted, sorted, duplicates, anything — which is exactly why it costs O(n) in the worst case: in the worst case the target is last, or missing entirely, and you have to look at every element. For a short list, or a list you're only going to search once, that's often the right answer; building a faster structure for a one-time lookup isn't worth it.

## Binary search: trading a sort for speed

If the data is already sorted, you can do dramatically better. Binary search looks at the middle element of the range. If it matches, you're done. If the target is smaller, everything from the middle to the end can be discarded — a sorted order guarantees it isn't there. If the target is larger, discard the bottom half instead. Either way, one comparison eliminates half the remaining candidates.

Trace it on `[2, 5, 8, 12, 16, 23, 38, 45, 56, 72]` searching for `23`: the middle index is 4 (0-indexed), value `16`. `23 > 16`, so discard indices 0–4, leaving `[23, 38, 45, 56, 72]`. New middle is `45`. `23 < 45`, so discard the top half, leaving `[23, 38]`. New middle (rounding down) is `23` — found, in three comparisons instead of the five linear search would have needed on average. Because each comparison halves the range, the number of comparisons needed is the number of times you can halve n before reaching 1, which is exactly log₂(n) — O(log n). The price is the sorted precondition: if the array isn't sorted, or gets new elements inserted between searches, you either have to re-sort it (O(n log n), which can erase the benefit for a single search) or maintain the sort as you go.

## Recursion: a function that calls itself

A recursive function solves a problem by handling a small piece of it directly and delegating the rest to a smaller copy of the same problem. Every correct recursive function needs two parts: a **base case**, small enough to answer without recursing, and a **recursive case**, which reduces the problem and calls itself on the reduced version. Factorial is the canonical example:

```
factorial(n):
  if n == 0: return 1        # base case
  return n * factorial(n-1)  # recursive case
```

`factorial(0)` returns `1` directly. `factorial(4)` calls `factorial(3)`, which calls `factorial(2)`, which calls `factorial(1)`, which calls `factorial(0)`, which finally hits the base case and returns `1`. Then each pending call multiplies in its own `n` as the stack unwinds: `1 → 1*1=1 → 2*1=2 → 3*2=6 → 4*6=24`.

## The call stack makes recursion work — and limits it

Every time a function calls another function (including itself), the running program pushes a **stack frame** holding that call's local variables, arguments, and the address to return to once it finishes. Recursive calls stack these frames up just like any other function call: `factorial(4)`'s frame is still sitting on the stack, incomplete, while it waits for `factorial(3)` to return a value it can multiply. That's why a missing or unreachable base case is dangerous in a way a normal bug isn't: the function keeps calling itself and pushing new frames with no path back, until the call stack — which has a fixed, finite size — runs out of room and the program crashes with a **stack overflow**. The same failure mode as an infinite loop, but the symptom is a crash rather than a hang, because the loop's "state" here is memory, not just time.

## Tail recursion: when the stack doesn't have to grow

Look again at `factorial`'s recursive case: `return n * factorial(n-1)`. After `factorial(n-1)` returns, there's still work left — the multiplication by `n` — before this call can return. That pending work is why its frame has to stay on the stack. Compare that to a tail-recursive version that carries the running product along as an argument instead:

```
factorial_tail(n, acc=1):
  if n == 0: return acc
  return factorial_tail(n-1, n*acc)
```

Here the recursive call is the absolute last thing the function does — there's no multiplication waiting for it to return, nothing left to compute. That's what "tail recursion" means: the recursive call is in tail position, with no pending work after it. Because the current frame has nothing left to do once it makes that call, some languages and compilers can recognize the pattern and reuse the current stack frame for the next call instead of pushing a new one — a technique called tail-call optimization — turning what looks like recursion into, mechanically, a loop that never grows the stack. Not every language does this (Python notably doesn't), which is why the same algorithm can recurse safely to a depth of a million in one language and overflow the stack at a few thousand in another — the difference isn't the algorithm, it's whether the runtime recognizes and optimizes the tail call.
