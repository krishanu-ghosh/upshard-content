---
title: "Aho-Corasick: Matching Thousands of Patterns in One Pass"
description: A trie with failure links generalizes KMP to search for many patterns against a text simultaneously, in one linear-time scan.
roadmap: datastructures-and-algorithms
node: aho-corasick-and-multi-pattern-matching
group: advanced-string-algorithms
video:
  provider: youtube
  id: "XWujo7KQL54"
  start: 0
  end: null
creator:
  name: ComputerBread
  channel_url: https://www.youtube.com/@ComputerBread
duration_seconds: 833
level: intermediate
prerequisites: ["suffix-trees-and-longest-common-substring"]
takeaways:
  - "Aho-Corasick finds every occurrence of every pattern from a fixed dictionary of k patterns in a text of length n in O(n + m + z) time, where m is the total length of all patterns and z is the number of matches — one pass over the text, regardless of how many patterns there are."
  - "The structure is a trie of all patterns plus failure links: each trie node gets a failure link to the longest proper suffix of its path that is also a prefix of some pattern in the dictionary, which is exactly KMP's failure function generalized from one pattern to a set of patterns."
  - "When a character mismatches during the scan, following a failure link resumes matching from the longest already-matched suffix instead of restarting from the trie root, so the text pointer never backtracks — this is what keeps the whole scan to one pass."
  - "Output (dictionary) links chain a node to the nearest ancestor via failure links that is itself the end of some pattern, so a single text position can report multiple pattern matches that end there (e.g. patterns 'he' and 'she' both ending at the same position) without walking the whole failure chain by hand each time."
  - "Real systems — spam and profanity filters, network intrusion detection signature matching, and DNA multi-motif search — all reduce to 'match thousands of fixed patterns against one stream' and use Aho-Corasick (or a close variant) instead of running a separate search per pattern."
check_questions:
  - q: "Why is Aho-Corasick described as a generalization of KMP rather than an unrelated algorithm?"
    a: "KMP's failure function tells you, on a mismatch, the longest proper prefix of the single pattern that is also a suffix of what you've matched so far, letting you resume without rescanning the text. Aho-Corasick builds the same kind of failure link, but for every node of a trie holding many patterns at once, so the 'resume point' logic generalizes from one pattern's prefix table to a shared automaton over the whole dictionary."
  - q: "If patterns 'he', 'she', and 'his' are all in the dictionary, why can one position in the text report more than one match, and how does the automaton find them without extra scanning?"
    a: "The trie node for 'he' lies on the path for 'she' too, so when the scan reaches the end of 'she' it has also just passed through the end of 'he' one step earlier in the trie structure; an output link from the 'she' node to the 'he' node (found by following failure links until hitting another pattern-end node) lets the algorithm report both matches at that text position by walking a short output-link chain, not by rescanning the text."
  - q: "Why would a system doing intrusion detection against thousands of attack signatures prefer Aho-Corasick over running Rabin-Karp or KMP once per signature?"
    a: "Running k independent pattern searches over the same text costs O(k * n) in the worst case, and that cost only grows as the signature list grows. Aho-Corasick builds one automaton from all k patterns up front and then scans the text exactly once, in O(n + m + z) total — the cost of adding more patterns shows up only in automaton construction (proportional to total pattern length), not in a multiplied per-pattern scan of the traffic."
deeper:
  - label: "Aho-Corasick Algorithm (cp-algorithms)"
    url: "https://cp-algorithms.com/string/aho_corasick.html"
related_formulas: []
---

## The problem one KMP scan can't solve

KMP, Rabin-Karp, and the Z-algorithm all answer the same question: does *this one pattern* occur in the text, and where? Multi-pattern matching asks a different question — given a dictionary of thousands of patterns, find every occurrence of every one of them in a single text, without re-scanning the text once per pattern. Running KMP `k` times costs O(k * n); for a spam filter checking a message against 50,000 banned phrases, or an intrusion-detection system checking network traffic against tens of thousands of attack signatures, that multiplier makes the naive approach unusable at scale. Aho-Corasick solves it in one pass, independent of `k`.

## Building the automaton: a trie plus failure links

**Step 1 — the trie.** Insert every pattern into a trie as usual, sharing common prefixes. If the dictionary is `{"he", "she", "his", "hers"}`, the paths for "he" and "hers" share the first two characters. Mark every node that completes a pattern.

**Step 2 — failure links.** This is the generalization of KMP's failure function. For every trie node `v` representing the string `x` (the path from the root to `v`), its failure link points to the trie node representing the longest proper suffix of `x` that is *also* a prefix of some pattern in the dictionary (i.e., also a path from the root). These links are computed with a breadth-first traversal of the trie: process nodes level by level, and for each child, follow the parent's failure link and see how far you can extend the match — exactly the same recurrence KMP uses to build its single-pattern failure table, just run over a trie instead of a line.

**Step 3 — output links.** Because one node's path can be a suffix of another pattern that completes earlier in the failure chain (as "he" is a suffix of "she"), you additionally chain nodes to the nearest pattern-ending ancestor along their failure links. This lets the scan report every pattern ending at a given text position in time proportional to the number of matches there, not the depth of the trie.

## Scanning the text

With the automaton built, you scan the text once, left to right, maintaining a "current node" pointer that starts at the root. For each character: if the current node has a child for that character, move to it. If not, follow failure links — exactly as KMP follows its failure function on a mismatch — until you find a node that does have that child, or you fall back to the root. At every node you land on, check (via the output link) whether it marks the end of one or more patterns, and if so, report them at the current text position. The key property is that the text pointer *never moves backward* — every character is examined, and the automaton state transitions absorb all the "what partial match do we still have" bookkeeping, so the whole scan is O(n) plus O(z) for reporting the `z` total matches, after O(m) to build the automaton from patterns totaling length `m`.

## Worked sketch

Dictionary `{"a", "ab", "bc", "bca", "c", "caa"}` against text `"abccab"`: the scanner walks `a` (matches pattern "a" — report), `b` (now at node "ab" — report "ab"), `c` (failure link bounces from "abc", which doesn't exist, back toward a node representing "bc" or "c" depending on how the dictionary's structure resolves — report whichever pattern-ends apply), and so on, never re-reading a character of the text. The point isn't to trace every transition by hand but to see that each step is O(1) amortized, regardless of how many patterns are loaded into the automaton.

## Where this shows up in production

Aho-Corasick (or a close variant, like the `aho-corasick` crate used inside ripgrep, or Snort's and Suricata's multi-pattern matchers) is the standard answer whenever a system needs to check one stream against a large, mostly-static set of patterns: spam and profanity filters scanning messages against banned-word lists, network intrusion detection systems matching packet payloads against thousands of attack signatures, antivirus engines matching files against malware byte signatures, and bioinformatics tools searching a genome for thousands of known motifs or primers simultaneously. In every case, the shared shape is the same one this lesson covers — a fixed dictionary, a long input stream, and a hard requirement to find every match in a single pass.
