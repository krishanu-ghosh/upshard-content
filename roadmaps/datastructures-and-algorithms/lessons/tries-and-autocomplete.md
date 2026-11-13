---
title: "Tries and Autocomplete"
description: Build a full autocomplete system on a trie with word-end markers and prefix word counts, and weigh the tradeoffs against a hash set.
roadmap: datastructures-and-algorithms
node: tries-and-autocomplete
group: more-advanced-structures
video:
  provider: youtube
  id: "us0qySiUsGU"
  start: 0
  end: null
creator:
  name: Tushar Roy - Coding Made Simple
  channel_url: https://www.youtube.com/@tusharroy2525
duration_seconds: 1182
level: intermediate
prerequisites: ["hash-tables-and-tries"]
takeaways:
  - "A full autocomplete trie needs two pieces of data per node beyond the character links: a flag marking whether a complete word ends there, and a count of how many words pass through that node."
  - "Inserting a word walks or creates one node per character and increments a per-node word count along the way, which is what lets prefix-count queries run in O(prefix length) instead of a tree scan."
  - "Finding every completion of a typed prefix is two phases: walk the prefix once (O(prefix length)), then depth-first search the subtree below that node to collect every word stored under it."
  - "A hash set can only tell you whether one exact string is a member; it has no way to answer 'how many stored words start with this prefix' without scanning every entry, which is why autocomplete is built on a trie instead."
  - "The tradeoff is space: a trie typically uses more memory than a hash set for the same word list because every shared prefix still needs one node per character, even though that's exactly what buys fast prefix operations."
  - "Ranking suggestions (e.g. by frequency or popularity) is usually layered on top by storing extra metadata — like a frequency count or a cached top-k list — at the nodes where words end or pass through."
check_questions:
  - q: "Why does a trie need a per-node word count, not just an end-of-word flag, to support 'how many words start with this prefix' efficiently?"
    a: "The end-of-word flag only tells you whether the node itself completes a word, not how many words continue past it in the subtree below. A running count incremented at every node a word passes through during insertion lets you read off, in O(1) after walking the prefix, how many total words share that prefix — without ever touring the subtree."
  - q: "Walking a prefix in a trie takes time proportional to the prefix's length. Why doesn't finding all completions of that prefix also take time proportional only to the prefix's length?"
    a: "Locating the prefix's node is O(prefix length), but after that you still need to discover and return every distinct word stored below that node, which requires visiting every node in that subtree — so listing k matching words of average length L costs roughly O(prefix length + k*L), not O(prefix length) alone. The prefix walk is cheap; enumerating the matches is proportional to how many matches there are."
  - q: "A dictionary app needs both 'is this exact word valid?' and 'what are all the completions of what the user has typed so far?'. Why not just use a hash set and accept a slower prefix search?"
    a: "A hash set gives O(1) exact-match lookups but has no structural notion of 'words that start with X' — answering a prefix query would mean iterating every entry in the set and checking each one, which is O(n) in the dictionary size regardless of how short the prefix is. A trie answers the same prefix query in time proportional to the prefix length plus the number of matches, which is what makes it the right structure whenever prefix queries are a first-class requirement rather than a rare afterthought."
deeper:
  - label: "Trie: Insert and Search (GeeksforGeeks)"
    url: "https://www.geeksforgeeks.org/dsa/trie-insert-and-search/"
related_formulas: []
---

## From "store strings" to "search by prefix"

A basic trie answers one question well: is this exact string present? Autocomplete needs more than that — given whatever a user has typed so far, it needs every stored word that continues it, and often a count of how many such words exist before even listing them. Building that on top of a trie means attaching two extra pieces of information to the structure you already know: an **end-of-word flag** on any node where a complete, inserted word terminates, and a **word count** at every node, tracking how many inserted words pass through it (including ones that end exactly there).

## Insertion with counting

Inserting a word walks from the root, one character at a time, creating a child node for any character that doesn't already have an edge, and following the existing edge where one does. The new piece: at every node visited along that path — including the root — increment a counter. By the time the last character is placed, every node on the path "knows" it has one more word passing through it than before, and the final node additionally gets its end-of-word flag set. Insert "car," "care," "cart," and "cat," and the node for "ca" ends up with a count of 4 (all four words pass through it), the node for "car" has a count of 3 ("car," "care," "cart") and its own end-of-word flag set (since "car" is itself a complete word), and the node for "cat" has a count of 1 and its end-of-word flag set.

## Answering "how many words start with this prefix?"

Because every node already tracks how many inserted words pass through it, answering "how many words start with 'ca'?" is just: walk the trie along "c" then "a" — two steps, O(prefix length) — and read the count stored at the node you land on. No scanning, no subtree exploration. This is the payoff for maintaining the counter during insertion: a query that would otherwise require inspecting every stored word becomes a single lookup proportional only to the prefix's length.

## Answering "what are the words that start with this prefix?"

Listing the actual completions is a two-phase operation. First, walk the prefix exactly as above to find the node representing it — O(prefix length). Second, run a depth-first search from that node downward, following every edge and recording a completed word any time you pass a node with its end-of-word flag set, reconstructing each word from the path taken. This phase's cost is proportional to the size of the subtree actually explored — roughly the number of matching words times their average length — not to the size of the whole dictionary. So the full query costs `O(prefix length + total characters in all matches)`, scaling with the size of the answer rather than the size of the dataset.

## Hash set vs. trie: the actual tradeoff

A hash set of the same words gives O(1) average-case membership testing — "is 'cart' in the dictionary?" — and uses memory roughly proportional to the total characters across all distinct stored words, since each string is stored once as an opaque blob. But it has no way to exploit shared prefixes: answering "how many words start with 'ca'?" means iterating the entire set and checking each entry's prefix by hand, which is O(n) in the number of stored words no matter how short the prefix is.

The trie flips that tradeoff. It typically costs more memory than a hash set for the same word list, because every node needs space for its children pointers (or a small array/map keyed by character) plus the count and flag — and critically, shared prefixes like "ca" in "car," "care," "cart," and "cat" are stored once, as one path, rather than four times over, but the per-node bookkeeping overhead still adds up, especially for sparse alphabets or Unicode. In exchange, prefix-based queries — count-with-prefix, list-all-completions, longest-common-prefix — become genuinely efficient instead of requiring a full scan. The decision rule is simple: if the application only ever needs exact lookups, a hash set is simpler and usually leaner; the moment prefix search is a real, repeated requirement — autocomplete, spell-check suggestions, IP routing tables — the trie's extra memory buys operations the hash set structurally cannot do cheaply. Production autocomplete systems often go further, caching a precomputed top-k list of popular completions at high-traffic nodes so even the DFS-collection phase is skipped for common prefixes, trading still more memory for even faster responses.
