---
title: "String Matching: Brute Force to Rabin-Karp"
description: Four ways to find a pattern inside a text — brute force, KMP, Boyer-Moore, and Rabin-Karp — plus suffix arrays for repeated queries.
roadmap: datastructures-and-algorithms
node: string-matching-and-suffix-arrays
group: advanced-string-algorithms
video:
  provider: youtube
  id: "H4VrKHVG5qI"
  start: 0
  end: null
creator:
  name: Tushar Roy - Coding Made Simple
  channel_url: https://www.youtube.com/@tusharroy2525
duration_seconds: 900
level: intermediate
prerequisites:
  - arrays-linked-lists-stacks-and-queues
takeaways:
  - Brute-force search checks every alignment of the pattern against the text, costing O(nm) time in the worst case because a near-miss can force it to backtrack character by character.
  - Knuth-Morris-Pratt precomputes a "failure function" over the pattern itself, so after a mismatch it knows how far it can safely skip without ever re-checking a text character it already matched.
  - Boyer-Moore scans the pattern right to left and uses a bad-character rule to jump the pattern past a mismatched text character entirely, which is why it often beats KMP in practice on natural-language text.
  - Rabin-Karp hashes the pattern once and then slides a rolling hash across the text, comparing hashes instead of characters and only checking a full match when hashes collide, which makes it a natural fit for finding multiple patterns at once.
  - A suffix array — the sorted list of every suffix of a string, stored as starting indices — turns repeated substring queries against the same text into binary searches, trading one upfront O(n log n) build for very fast lookups afterward.
check_questions:
  - q: Why can brute-force string search degrade to O(nm) time instead of the O(n) you might hope for?
    a: A pathological text and pattern (like searching for "aaaab" inside "aaaaaaaaaa...") can match the first m-1 characters at every single alignment before failing, so the mismatch check itself costs O(m) work at up to n different starting positions.
  - q: What does KMP's failure function actually tell the algorithm when a mismatch happens?
    a: It tells the algorithm the length of the longest proper prefix of the pattern that is also a suffix of the part already matched, so it can resume comparing from that point instead of restarting the pattern from its first character.
  - q: Why is a rolling hash central to Rabin-Karp's speed?
    a: It lets the algorithm compute the hash of the next window of text from the previous window's hash in constant time, rather than re-hashing the whole window from scratch at every shift.
deeper:
  - label: "ADS1: Boyer-Moore basics"
    url: "https://www.youtube.com/watch?v=4Xyhb72LCX4"
  - label: "Suffix array introduction"
    url: "https://www.youtube.com/watch?v=zqKlL3ZpTqs"
related_formulas: []
---

## The problem: find a needle in a haystack, fast

String matching asks a simple question — does pattern `P` of length `m` occur inside text `T` of length `n`, and if so, where? It sounds trivial until `T` is a genome with three billion characters, a codebase with a million lines, or a search index with the entire web behind it. The naive approach works but doesn't scale, so four classic algorithms trade different amounts of preprocessing for different amounts of speed.

## Brute force: the obvious starting point

The simplest approach slides the pattern across the text one position at a time and, at each position, compares characters left to right until either the whole pattern matches or a mismatch is found. Searching for `"abab"` in `"ababababab"` works fine this way. But searching for `"aaaab"` in a long run of `a`s is the worst case: at nearly every position, the algorithm matches four characters before the fifth one fails, so the total work approaches `O(n·m)` instead of the `O(n)` you'd want. The three algorithms below exist specifically to avoid repeating work the brute-force approach throws away.

## Knuth-Morris-Pratt: never recheck a text character

KMP's insight is that the pattern itself contains information about how to recover from a mismatch. Before searching, it builds a "failure function" (also called the prefix function) over the pattern: for every prefix of the pattern, how long is the longest proper prefix of *that* prefix which is also a suffix of it? When a mismatch happens after matching `k` characters, the failure function tells KMP exactly how many of those `k` characters it can skip re-verifying, because it already knows they match a shorter prefix of the pattern. The text pointer never moves backward, which guarantees `O(n + m)` total time — linear, no matter how adversarial the input.

## Boyer-Moore: skip using the mismatch itself

Boyer-Moore compares the pattern against the text right-to-left instead of left-to-right, and when a mismatch occurs, it uses the *mismatched text character* to decide how far to shift the pattern — this is the bad-character rule. If the text character doesn't appear in the pattern at all, the whole pattern can jump past it entirely. If it does appear, the pattern shifts to align that occurrence with the mismatch. On text with a large alphabet (like English prose, as opposed to DNA's four-letter alphabet), these jumps are often large, which is why Boyer-Moore is the algorithm behind many real-world text editors' "find" features — it frequently beats KMP in practice even though both are worst-case linear-ish.

## Rabin-Karp: compare hashes, not characters

Rabin-Karp takes a different angle: hash the pattern once, then compute a rolling hash of every length-`m` window of the text, sliding one character at a time. A rolling hash is designed so that the hash of the next window can be computed from the current window's hash in `O(1)`, typically by subtracting the outgoing character's contribution and adding the incoming one (treating the window as a number in some base). Most windows will have a different hash from the pattern and are rejected instantly; only a hash collision triggers an actual character-by-character comparison to rule out a false positive. This hash-first approach generalizes naturally to searching for *many* patterns at once — hash all of them, then check each window's hash against the whole set — which single-pattern algorithms like KMP and Boyer-Moore don't do as gracefully.

## Suffix arrays: pay once, query forever

All four algorithms above answer one query: does this one pattern appear in this one text? If you need to answer *many* substring queries against the *same* text — does "science" appear? does "comp" appear, and how many times? — repeating any of them from scratch each time wastes the fact that the text hasn't changed. A suffix array precomputes the sorted order of every suffix of the text (the suffix starting at index 0, at index 1, and so on), stored compactly as a list of starting positions. Because the suffixes are sorted, any substring query becomes a binary search over that sorted list — `O(log n)` per query after an `O(n log n)` build — which is why suffix arrays sit behind full-text search, genome alignment tools, and plagiarism detectors that need to answer the same kind of question over and over against a fixed body of text.
