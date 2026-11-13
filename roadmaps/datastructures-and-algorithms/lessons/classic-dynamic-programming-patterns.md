---
title: "Classic Dynamic Programming Patterns: LCS, LIS, and Interval DP"
description: The DP table shapes beyond knapsack — LCS and LIS table-filling, and interval DP over ranges — and how to spot which shape a problem needs.
roadmap: datastructures-and-algorithms
node: classic-dynamic-programming-patterns
group: more-interview-patterns
video:
  provider: youtube
  id: "Ua0GhsJSlWM"
  start: 0
  end: null
creator:
  name: NeetCode
  channel_url: https://www.youtube.com/@NeetCode
duration_seconds: 1105
level: intermediate
prerequisites:
  - bitmask-dynamic-programming
  - dynamic-programming-and-np-hard-problems
takeaways:
  - "Longest Common Subsequence (LCS) uses a 2D table dp[i][j] = the LCS length of the first i characters of one string and the first j of another; if the characters match, dp[i][j] = dp[i-1][j-1] + 1, otherwise dp[i][j] = max(dp[i-1][j], dp[i][j-1])."
  - "Longest Increasing Subsequence (LIS) uses a 1D table dp[i] = the length of the longest increasing subsequence ending exactly at index i, computed by checking every earlier index j and taking dp[j] + 1 wherever nums[j] < nums[i] — O(n^2), with a well-known O(n log n) upgrade using binary search over a 'smallest tail' array."
  - "Interval (range) DP solves a problem over every contiguous subrange of a sequence with dp[i][j] = the answer for the range from index i to index j, filled in order of increasing range length rather than increasing index, since dp[i][j] depends on narrower ranges strictly inside it."
  - "Longest Palindromic Subsequence is the canonical interval DP: dp[i][j] = dp[i+1][j-1] + 2 when the end characters match, otherwise max(dp[i+1][j], dp[i][j-1]) — which only makes sense if the inner range [i+1][j-1] is already filled, forcing the length-first fill order."
  - "The question that tells you which shape to reach for is 'what does one subproblem need to know': two independent prefixes (LCS-style 2D table), one prefix ending at a specific point (LIS-style 1D table), or one specific contiguous range (interval DP) — the recurrence's dependencies dictate both the table's dimensions and the order you must fill it in."
check_questions:
  - q: "Why does the LCS recurrence branch into two different cases depending on whether the current characters match?"
    a: "Because a matching pair of characters can only ever help by being included in the common subsequence together, so dp[i][j] = dp[i-1][j-1] + 1 extends the best answer from strictly before both characters; but when they don't match, at most one of the two current characters can be in the LCS, so the best answer is the better of 'drop this character from the first string' or 'drop this character from the second string' — max(dp[i-1][j], dp[i][j-1]) — since you can't yet know which one it's better to discard."
  - q: "In Longest Increasing Subsequence, why is dp[i] defined as 'the LIS ending exactly at index i' rather than 'the LIS using only the first i elements'?"
    a: "Because an increasing subsequence's validity depends on the actual value of its last element, not just how many elements have been considered so far — defining dp[i] as ending exactly at i lets the recurrence check 'can nums[i] legally extend the subsequence that ends at j' via nums[j] < nums[i], which wouldn't be answerable from a prefix-only definition that doesn't track where each candidate subsequence currently ends."
  - q: "Why must interval DP tables be filled by increasing range length rather than by increasing row index i?"
    a: "Because dp[i][j] for the range [i, j] depends on dp[i+1][j-1], dp[i+1][j], and dp[i][j-1] — all strictly narrower ranges nested inside [i, j] — so every one of those must already be computed before dp[i][j] can be; filling by increasing length guarantees every shorter (and therefore already-needed) range is finished before any longer range that depends on it is attempted, which a simple row-by-row or column-by-column fill order does not guarantee."
deeper:
  - label: "DP 28: Longest Palindromic Subsequence (interval DP walkthrough)"
    url: "https://www.youtube.com/watch?v=6i_T5kkfv4A"
related_formulas: []
---

## Beyond knapsack: recognizing a DP problem by its table shape

The general DP vocabulary — overlapping subproblems, memoization vs. tabulation, the 0/1 knapsack and bitmask-DP shapes from earlier lessons — tells you *that* a problem has optimal substructure. It doesn't tell you *what the table should look like*. In practice, a huge share of interview DP problems fall into one of three recurring table shapes, and recognizing which shape fits is most of the battle.

This lesson covers the other three big ones: a 2D table over two sequences (LCS), a 1D table over one sequence with an "ending at i" definition (LIS), and a 2D table over ranges of one sequence (interval DP).

## Shape 1: two-sequence table — Longest Common Subsequence

Given two strings, `dp[i][j]` holds the length of the longest common subsequence between the first `i` characters of string A and the first `j` characters of string B.

Walk `A = "ABC"`, `B = "AC"`. Build a (4×3) table indexed by `i = 0..3`, `j = 0..2`, with row/column 0 all zero (empty prefix has no common subsequence).

- At `i=1, j=1` ('A' vs 'A'): match, so `dp[1][1] = dp[0][0] + 1 = 1`.
- At `i=2, j=1` ('B' vs 'A'): no match, so `dp[2][1] = max(dp[1][1], dp[2][0]) = 1`.
- At `i=3, j=2` ('C' vs 'C'): match, so `dp[3][2] = dp[2][1] + 1 = 2`.

The final answer, `dp[3][2] = 2`, matches the actual LCS "AC". A match extends diagonally because a matched pair can only help by being used *together*. A mismatch takes the max of its neighbors because at most one of the two current characters survives into the answer, and the recurrence can't yet know which one — so it keeps the better of discarding either.

## Shape 2: single-sequence, "ending at i" table — Longest Increasing Subsequence

LIS looks like it should be a simple prefix table: "the longest increasing subsequence among the first i elements." That definition can't support a correct recurrence, though, because knowing *how long* the best subsequence is doesn't tell you whether a new, bigger element can extend it. The fix is to define `dp[i]` as the length of the longest increasing subsequence that *ends exactly at index i*.

For `[1, 3, 2, 4]`:

- `dp[0] = 1` (just `[1]`).
- `dp[1]`: check `j=0`, `nums[0]=1 < nums[1]=3`, so `dp[1] = dp[0]+1 = 2` (`[1,3]`).
- `dp[2]`: check `j=0` (`1 < 2`, candidate `dp[0]+1=2`) and `j=1` (`3 < 2` fails). Best is `2` (`[1,2]`).
- `dp[3]`: check `j=0` (candidate 2), `j=1` (candidate `dp[1]+1=3`), `j=2` (candidate `dp[2]+1=3`). Best is `3` (`[1,2,4]` or `[1,3,4]`).

The answer is `max(dp) = 3`. This is O(n²) because every index checks every earlier index. The well-known O(n log n) upgrade replaces that inner scan with binary search over an auxiliary array of "smallest possible tail value for an increasing subsequence of each length" — but the O(n²) version is what demonstrates the actual DP shape.

## Shape 3: range table — interval DP via palindromic subsequence

Some problems aren't naturally about prefixes at all. They're about *every contiguous range* of a sequence, and the answer for a range depends on the answers for narrower ranges strictly inside it.

Longest Palindromic Subsequence is the standard teaching example: `dp[i][j]` is the length of the longest palindromic subsequence within `s[i..j]`. If `s[i] == s[j]`, those two matching ends can both be used, wrapping a palindrome found inside: `dp[i][j] = dp[i+1][j-1] + 2`. If they don't match, the answer is the better of dropping one end: `dp[i][j] = max(dp[i+1][j], dp[i][j-1])`.

For `s = "BBAB"` (indices 0-3):

- Base case: every single character is a palindrome of length 1 (`dp[i][i] = 1`).
- Length-2: `dp[0][1]` ("BB") — ends match, so `dp[0][1] = dp[1][0]+2 = 0+2 = 2` (inner range is empty). `dp[2][3]` ("AB") — no match, so `dp[2][3] = max(dp[3][3], dp[2][2]) = 1`.
- Length-3: `dp[1][3]` ("BAB") — ends match (`B`, `B`), so `dp[1][3] = dp[2][2] + 2 = 3`.
- Length-4: `dp[0][3]` ("BBAB") — ends match, so `dp[0][3] = dp[1][2] + 2`. `dp[1][2]` ("BA") has no match, so it's `max(dp[2][2], dp[1][1]) = 1`. Total: `dp[0][3] = 3`.

The answer, 3, matches the actual longest palindromic subsequence present in "BBAB": "BAB".

The critical implementation detail is fill order. Because `dp[i][j]` always depends on strictly shorter ranges, the table must be filled by increasing range length — all length-1 ranges, then all length-2, and so on — not row by row. A row-by-row fill would reach `dp[i][j]` before `dp[i+1][j-1]` is ready whenever `j - i` is large.

## The checklist

When a new problem feels DP-shaped, ask what one subproblem's answer needs to know.

- Two independent positions in two different sequences → LCS-style 2D table over `(i, j)` pairs from two inputs.
- One position in one sequence, where validity depends on what the subsequence currently ends with → LIS-style 1D table defined by "ending at i," filled left to right.
- A specific contiguous span of one sequence, where the answer composes from spans nested inside it → interval DP, filled by increasing length.

Matching the recurrence's actual dependencies to one of these three shapes is what turns "this smells like DP" into a table you can actually fill in correctly.
