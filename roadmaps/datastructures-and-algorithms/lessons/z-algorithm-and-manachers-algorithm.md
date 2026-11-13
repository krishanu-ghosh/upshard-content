---
title: "The Z-Array and Manacher's Algorithm: Two More Linear-Time String Tricks"
description: Two independent O(n) ideas — the Z-array for pattern matching without a failure function, and Manacher's algorithm for the longest palindromic substring.
roadmap: datastructures-and-algorithms
node: z-algorithm-and-manachers-algorithm
group: advanced-string-algorithms
video:
  provider: youtube
  id: "CpZh4eF8QBw"
  start: 0
  end: null
creator:
  name: Tushar Roy - Coding Made Simple
  channel_url: https://www.youtube.com/@tusharroy2525
duration_seconds: 1442
level: intermediate
prerequisites: [string-matching-and-suffix-arrays]
takeaways:
  - "The Z-array at index i holds the length of the longest substring starting at i that matches the string's own prefix — a different bookkeeping idea from KMP's failure function, even though both achieve O(n) matching."
  - "Z-array pattern matching concatenates pattern + separator + text into one string; any index in the text region where the Z-value equals the pattern length marks a match, found in a single O(n + m) pass."
  - "The Z-array is built in O(n) by maintaining a window [L, R] of the rightmost match found so far and reusing previously computed Z-values inside that window instead of recomparing characters from scratch."
  - "Manacher's algorithm finds the longest palindromic substring in O(n) by transforming the string (inserting separators between characters) so every palindrome, odd or even length, has a single center."
  - "Manacher's algorithm reuses a mirrored palindrome's radius within the current rightmost palindrome's span instead of re-expanding from scratch at every center, which is what turns an O(n^2) expand-around-center approach into O(n)."
  - "Both algorithms are 'two pointers plus a window of already-proven information' in disguise — they avoid redundant character comparisons by exploiting a symmetry (prefix-repetition for Z, palindrome mirroring for Manacher) instead of hashing or automaton states."
check_questions:
  - q: "Why does computing the Z-array for pattern$text solve pattern matching, and why is the separator character necessary?"
    a: "Z-values measure overlap with the whole string's prefix, which is the pattern; any position in the text whose Z-value equals the pattern's length has the full pattern starting there. The separator (a character that appears in neither string) prevents a match from spilling across the pattern/text boundary and inflating a Z-value with characters that aren't really part of the text."
  - q: "In Manacher's algorithm, when you're expanding the palindrome centered at i and i falls inside the current rightmost palindrome [L, R], why can you initialize its radius using the mirror position instead of starting from 0?"
    a: "Because the substring inside [L, R] is itself a palindrome, the palindrome centered at the mirror of i (reflected across the center of [L, R]) has already been computed, and by symmetry the palindrome at i must extend at least that far, unless the mirrored palindrome touches the boundary L — in that case it might extend further to the right, so you still have to verify by expansion beyond R, but you never recheck the part already guaranteed by symmetry."
  - q: "Both the Z-array and KMP's failure function describe prefix behavior, so why doesn't the Z-array just replace KMP in practice?"
    a: "They encode the same kind of information from different reference points — Z[i] measures overlap with the prefix of the whole string, while KMP's table measures overlap with the prefix for every proper prefix the pattern — so the Z-array makes pattern-anywhere-in-text matching very direct via concatenation, but KMP's failure function is more natural for streaming text one character at a time without ever materializing a combined string, which matters when the text is huge or arrives online."
deeper:
  - label: "Longest Palindromic Substring — Manacher's Algorithm (Tushar Roy)"
    url: "https://www.youtube.com/watch?v=V-sEwsca1ak"
related_formulas: []
---

## Two tricks, one family resemblance

You already know KMP uses a failure function and Rabin-Karp uses rolling hashes to avoid rechecking characters you've already compared. The Z-algorithm and Manacher's algorithm solve different problems the same way: they both maintain a *window of already-verified information* and reuse it instead of starting every comparison from scratch. They don't share code or data structures, but they share that one idea, which is worth internalizing because it reappears constantly in string processing.

## The Z-array

For a string `S` of length `n`, the Z-array is defined by `Z[i]` = the length of the longest substring starting at position `i` that is also a prefix of `S`. By convention `Z[0]` is undefined (or set to 0 or n, implementations vary). Take `S = "aabxaabxcaabxaabxay"`. At every position where `S` restarts matching `"aab..."`, you get a nonzero Z-value; everywhere else it's 0 or small.

**Why this gives you pattern matching for free.** To find all occurrences of a pattern `P` (length `m`) in text `T` (length `n`), build the string `P + "$" + T` (`$` being a separator byte that appears in neither string) and compute its Z-array. Any position `i` in the text portion with `Z[i] == m` marks an occurrence of `P` starting at that offset into `T`, because a Z-value of exactly `m` means the substring there matches the full prefix, which is exactly `P`. This is a clean O(n + m) algorithm that needs no failure-function backtracking logic at match time — you just scan the Z-array for `m`s.

**Building it in O(n).** The naive way — for each `i`, compare character by character against the prefix — is O(n^2) in the worst case. The linear algorithm keeps a window `[L, R]` representing the rightmost prefix-match box found so far (i.e., `S[L..R]` is known to equal `S[0..R-L]`). When computing `Z[i]` for `i > R`, you have no information, so you expand naively and update `[L, R]`. But when `i` falls inside `[L, R]`, you know `S[i..R]` matches some chunk of the prefix already, specifically the chunk at `S[i-L..R-L]` — so you copy `Z[i-L]` as a starting guess (clamped to not overrun `R`), and only expand past `R` by direct comparison if needed. Every character gets compared at most a constant number of times across the whole run, giving O(n) total.

## Manacher's algorithm

The longest palindromic substring problem has an obvious O(n^2) solution: expand outward from every possible center (there are `2n - 1` of them, counting between-character gaps for even-length palindromes) and see how far it stays a palindrome. Manacher's algorithm makes this O(n).

**The transform.** First, insert a separator (say `#`) between every character and at both ends: `"abba"` becomes `"#a#b#b#a#"`. Now every palindrome in the transformed string — odd or even in the original — has an odd length and a single center character. This sidesteps having to special-case even-length palindromes separately.

**The reuse trick.** Scan left to right, maintaining the rightmost palindrome found so far, described by its center `C` and right edge `R`. For the next center `i`, if `i < R`, then `i` has a mirror position `i' = 2C - i` inside the already-explored palindrome. Because the region around `C` is itself a palindrome, the palindrome radius at `i` must be *at least* as large as `min(R - i, radius[i'])` — you get that much for free by symmetry, no comparisons needed. You only start comparing characters once you try to expand past `R`, and because `R` only ever moves rightward and each comparison that extends it is "new" work, the total number of character comparisons across the entire scan is O(n).

**Worked snippet.** For `"#a#b#a#"` (from `"aba"`), the center index 3 (the middle `b`) mirrors around nothing interesting since it's the first real center explored, but once you reach a later center whose mirror falls inside a previously found palindrome, you skip straight to the pre-computed radius instead of re-expanding character by character — that skip is the entire point.

## Why learn both together

Neither algorithm depends on the other, but they're taught together because they're the two classic examples of "exploit a self-similarity in the string to avoid redundant work," distinct from the failure-function and hashing tricks you already know. Once you've internalized this pattern, spotting it in new problems — compressed strings, run-length matching, circular string rotations — becomes much easier.
