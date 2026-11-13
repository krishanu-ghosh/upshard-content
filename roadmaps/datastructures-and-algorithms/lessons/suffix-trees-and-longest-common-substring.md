---
title: "Suffix Trees: Ukkonen's Algorithm and the Longest Common Substring"
description: Suffix trees as a distinct structure from suffix arrays — Ukkonen's construction, conceptually, and generalized trees for cross-string matching.
roadmap: datastructures-and-algorithms
node: suffix-trees-and-longest-common-substring
group: advanced-string-algorithms
video:
  provider: youtube
  id: "aPRqocoBsFQ"
  start: 0
  end: null
creator:
  name: Tushar Roy - Coding Made Simple
  channel_url: https://www.youtube.com/@tusharroy2525
duration_seconds: 4572
level: intermediate
prerequisites: ["z-algorithm-and-manachers-algorithm"]
takeaways:
  - "A suffix tree is a compressed trie of all suffixes of a string, with edges labeled by substrings (represented as index ranges, not copied text) rather than single characters — this is the structural difference from a suffix array, which is just a sorted list of suffix start positions."
  - "Suffix trees support substring search in O(m) (just walk the tree by the pattern's characters), while a suffix array needs an O(m log n) binary search or extra LCP-array machinery to match that — the tree pays for this with higher memory overhead per node."
  - "Ukkonen's algorithm builds the suffix tree online, left to right, in O(n) total time, by maintaining an 'active point' and using suffix links to jump between related insertion points instead of restarting the walk from the root for every new suffix."
  - "Ukkonen's algorithm gets its linear bound from three tricks: representing edge labels as index ranges so extending all of them is O(1) (the 'trick' of an implicit global end), suffix links that turn repeated root-to-point walks into O(1) jumps, and skipping work once a suffix is already implicitly present in the tree."
  - "A generalized suffix tree built over S1 + '#' + S2 + '$' (distinct terminators) lets you find the longest common substring of two strings in O(n + m): find the deepest internal node whose subtree contains leaves from both strings, and its string-depth is the answer."
  - "Suffix trees generalize past two strings — a generalized suffix tree over any number of documents, each leaf tagged with its source, underpins full-text search engines and plagiarism/duplicate-content detection at scale."
check_questions:
  - q: "A suffix array plus an LCP array can answer most of the same queries as a suffix tree. What does the suffix tree give you more directly, and at what cost?"
    a: "The suffix tree gives you direct O(m) substring existence/location queries by walking edges character-by-character from the root, and the tree's internal-node structure directly encodes shared-prefix relationships (an internal node's string-depth is the length of the longest common prefix of all suffixes below it) without needing a separate LCP array or range-minimum-query structure. The cost is memory: each node needs child pointers (or a map) per branching character, and real implementations typically use several times the memory of an equivalent suffix array."
  - q: "Why is a suffix link ('the node for suffix S[i+1..n]' reachable directly from 'the node for suffix S[i..n]') the key to Ukkonen's linear time bound?"
    a: "Without suffix links, inserting each successive suffix into the tree would require walking down from the root again, which costs O(n) per suffix and O(n^2) overall. Suffix links let the algorithm jump directly to the insertion point for the next suffix from the previous one, so the total work across all insertions amortizes to O(n) instead of O(n) per suffix."
  - q: "For finding the longest common substring of two strings with a generalized suffix tree, why does the answer come from the deepest internal node with leaves from BOTH strings, rather than just the deepest internal node overall?"
    a: "An internal node's string-depth represents a substring shared by every suffix in its subtree, but that's only a substring common to both input strings if at least one leaf in the subtree traces back to each string; the single deepest internal node overall might have every leaf from only one of the two strings, meaning that substring doesn't appear in the other string at all. Coloring leaves by source and requiring both colors beneath a node is what restricts the search to genuinely shared substrings."
deeper:
  - label: "Suffix Tree. Ukkonen's Algorithm construction (cp-algorithms)"
    url: "https://cp-algorithms.com/string/suffix-tree-ukkonen.html"
related_formulas: []
---

## Not the same thing as a suffix array

You already know suffix arrays: sort every suffix of a string, store just the starting indices, and binary-search for a pattern. A suffix **tree** is a different structure that encodes the same information — every suffix of a string — but as an actual tree rather than a sorted list. Specifically, it's a compressed trie: take all `n` suffixes, insert them into a trie, then collapse every chain of single-child nodes into one edge labeled with a substring. Because it's compressed, the tree has only O(n) nodes despite the suffixes themselves totaling O(n^2) characters — each edge stores a label as a pair of indices into the original string, not a copy of the text.

The practical difference from a suffix array shows up at query time. Searching for a pattern of length `m` in a suffix tree means walking down from the root, consuming `m` characters of the pattern against edge labels — O(m) total, full stop. A suffix array needs binary search over sorted suffixes, each comparison potentially costing O(m), for O(m log n) — faster to build and far more memory-compact, but structurally less direct.

## Ukkonen's algorithm, conceptually

Building a suffix tree by inserting suffixes one at a time, naively, costs O(n^2) — insert suffix `i` by walking from the root, which costs up to O(n) each, `n` times. Ukkonen's algorithm gets this down to O(n) total, and it's worth understanding the three ideas that make that possible, without working through the full implementation (which is notoriously fiddly):

1. **Edges store index ranges, with an implicit "open" end for the growing boundary.** When you add a new character to the text, every leaf edge needs to conceptually extend by one character. Instead of actually touching every leaf, you let each leaf's edge store an end pointer as "the current end of the text," a single shared value. Bumping one global variable extends every leaf edge simultaneously — that's O(1) instead of O(current number of leaves).

2. **Suffix links connect related positions, so you never re-walk from the root.** For an internal node representing substring `xA` (where `x` is a single character and `A` a possibly-empty string), its suffix link points to the node representing `A`. When the algorithm finishes processing one suffix and moves to the next (which is the previous suffix with its first character chopped off), it follows the suffix link to jump almost directly to where the next insertion needs to happen, rather than starting from the root. This is what turns "O(n) insertions each costing up to O(n)" into amortized O(n) total.

3. **Rule 3 extensions stop early.** When extending the tree for a new character, if the character you're adding is already implicitly present along the path you'd be extending, the algorithm does nothing and moves on — the suffix is already represented without adding anything. This "showstopper" rule is part of why the total work stays bounded.

The textbook worked example is building the tree for `"abcabxabcd$"` character by character and watching the active point hop around via suffix links — it's the standard way people build intuition, and it's worth stepping through by hand once, even though you'd never hand-implement Ukkonen's algorithm in practice (library implementations or simpler O(n log n) alternatives like suffix-array-plus-LCP are far more common in real code).

## Generalized suffix trees and the longest common substring

A generalized suffix tree extends the idea to multiple strings: concatenate `S1 + "#" + S2 + "$"` using two distinct terminator characters that appear nowhere else, then build one suffix tree over the combined string. Each leaf is tagged with which original string it came from (based on which side of the terminators its suffix started on).

To find the **longest common substring** of `S1` and `S2`: every internal node's string-depth (the total length of the path from the root to that node) represents a substring common to every suffix passing through it. Color each leaf by its source string, then find the deepest internal node whose subtree contains leaves of both colors — that node's string-depth is the length of the longest substring that appears in both `S1` and `S2`, and walking the path to it reconstructs the substring itself. This runs in O(n + m) after construction, a dramatic improvement over the O(n * m) dynamic-programming table you'd otherwise use.

This generalizes further: a generalized suffix tree over any number of documents, each leaf tagged with its source, is the backbone of plagiarism detectors, duplicate-content detection in search engines, and genome comparison tools that need to find shared motifs across many sequences at once.
