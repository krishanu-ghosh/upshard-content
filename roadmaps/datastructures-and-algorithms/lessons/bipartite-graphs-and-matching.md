---
title: "Bipartite Graphs & Matching"
description: Testing whether a graph splits into two independent sides, then pairing up nodes across that split as efficiently as possible.
roadmap: datastructures-and-algorithms
node: bipartite-graphs-and-matching
group: union-find-and-graph-structure
video:
  provider: youtube
  id: "ory4WMX0rDU"
  start: 0
  end: null
creator:
  name: stoleemath
  channel_url: https://www.youtube.com/@stoleemath
duration_seconds: 446
level: intermediate
prerequisites:
  - union-find-and-disjoint-sets
takeaways:
  - "A graph is bipartite exactly when its vertices can be split into two groups such that every edge crosses between groups and none stays within one — equivalently, when it contains no odd-length cycle."
  - "Bipartiteness is tested with a 2-coloring BFS/DFS: color the start node red, color every neighbor the opposite color, and declare the graph non-bipartite the instant an edge connects two same-colored nodes."
  - "A matching is a set of edges with no shared endpoints; a maximum matching is the largest such set possible, which for a bipartite graph pairs up as many left-side and right-side vertices as the structure allows."
  - "An augmenting path starts and ends at unmatched vertices and alternates unmatched, matched, unmatched, matched edges; flipping every edge's status along it grows the matching by exactly one pair."
  - "Berge's theorem makes this a clean stopping rule: a matching is maximum if and only if no augmenting path exists relative to it, which is what repeated augmenting-path search (and its faster cousin, Hopcroft-Karp) is built around."
  - "Real assignment problems — jobs to applicants, shifts to workers, ads to slots — are bipartite matching once you draw 'can be assigned to' as an edge between the two sides."
check_questions:
  - q: "Why does a single odd-length cycle anywhere in a graph make it impossible to 2-color, and therefore non-bipartite?"
    a: "Walking around a cycle, 2-coloring forces colors to alternate at every step; after an even number of edges you're back to the starting color, which is consistent, but after an odd number of edges you'd need the last node to be both the opposite color of its predecessor and the same color as the start — a contradiction that only odd cycles produce."
  - q: "Why does finding one augmenting path and flipping its edges actually increase the matching size, rather than just rearranging it?"
    a: "An augmenting path has one more unmatched edge than matched edge (it starts and ends on unmatched edges, alternating throughout), so flipping every edge's status — matched becomes unmatched and vice versa — converts that one extra unmatched edge into a matched one while preserving the alternation elsewhere, netting exactly one additional matched pair with no endpoint used twice."
  - q: "Why can't maximum bipartite matching be solved by just greedily matching any available edge one at a time?"
    a: "A greedy pass can lock in an edge that blocks a much better overall pairing — e.g., matching A to X first when A could only ever match X, while B could match either X or Y, leaves B unmatched if greedy happens to pick wrong; augmenting-path search fixes this by explicitly un-matching and re-matching along a path whenever a better overall arrangement is found, something a single greedy pass can never undo."
deeper:
  - label: "The Augmenting Path Algorithm (Example)"
    url: "https://www.youtube.com/watch?v=C9c8zEZXboA"
related_formulas: []
---

## Splitting a graph in two

A graph is bipartite when its vertex set divides into two sides, left and right, such that every edge connects a left vertex to a right vertex — never left-to-left or right-to-right. Plenty of real structures are naturally bipartite: students and courses, jobs and applicants, ingredients and recipes that use them. The question "is this graph bipartite?" and the question "can I 2-color it so every edge joins different colors?" are the same question.

Testing it is a traversal you already know with one new rule. Pick any unvisited vertex, color it red, and do a BFS or DFS from it. Every time you move across an edge, color the neighbor the opposite color of the current node — unless that neighbor is already colored, in which case check it: if it's already the opposite color, fine, continue; if it's the *same* color as the current node, you've found an edge connecting two same-colored vertices, which is a contradiction, and the graph is not bipartite. Run this from every unvisited vertex to cover disconnected graphs. The reason this exactly detects bipartiteness is a clean equivalence: a graph is bipartite if and only if it has no cycle of odd length, and an odd cycle is precisely what forces the coloring contradiction — walk around it and the alternating color pattern can't close consistently.

## Matching: pairing across the split

Once you know a graph is bipartite, a natural next question is how to pair up vertices across the two sides — a *matching* is a subset of edges where no vertex appears in more than one chosen edge. A *maximum matching* is the largest matching the graph admits. This is the "assignment problem" in disguise: left side is applicants, right side is jobs, an edge means "qualified for," and a maximum matching is the most people you can place.

## Growing a matching one augmenting path at a time

Start from any matching, even the empty one. An *augmenting path* is a path that starts at an unmatched vertex, ends at a different unmatched vertex, and strictly alternates between edges not in the matching and edges in the matching. Because it starts and ends on "not in matching" edges and alternates, it always contains exactly one more unmatched edge than matched edge.

Flip every edge's status along that path — unmatched edges become matched, matched edges become unmatched — and the result is still a valid matching (every vertex along the path still touches at most one matched edge), but now with one additional pair matched overall, since the flip converted a net surplus of one unmatched edge into a matched one.

Worked example: left {A, B}, right {X, Y}, edges A-X, A-Y, B-X. Start with matching {A-X}. B is unmatched; look for an augmenting path from B: B-X (unmatched edge, since B-X isn't in the matching) reaches X, which *is* matched (to A) via edge A-X (matched edge), reaching A, which has another unmatched edge A-Y reaching Y, which is unmatched — path found: B-X-A-Y, alternating unmatched/matched/unmatched. Flip it: B-X becomes matched, A-X becomes unmatched, A-Y becomes matched. New matching: {B-X, A-Y} — size 2, up from 1, and nobody is double-booked.

Repeat this search-and-flip process until no augmenting path can be found from any unmatched vertex. Berge's theorem guarantees that's exactly when the matching is maximum — there's no cleverer rearrangement left to try. The naive version of this (BFS for one augmenting path at a time) runs in O(V·E); Hopcroft-Karp speeds this up to O(E√V) by finding a whole batch of shortest, vertex-disjoint augmenting paths in each round instead of just one, cutting the number of rounds needed to O(√V).

## Where this shows up

Any "assign these to those, respecting eligibility" problem is bipartite matching: scheduling workers to shifts, students to dorm rooms, ads to available slots, tasks to machines. It's also the combinatorial core of the stable matching framing (stable marriage / hospital-residents problems), though those add preference rankings on top of plain eligibility — matching answers "can everyone be paired at all," stable matching additionally asks "can it be done so nobody wants to defect."
