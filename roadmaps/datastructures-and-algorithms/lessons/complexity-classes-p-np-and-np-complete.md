---
title: "Complexity Classes: P, NP, and NP-Complete"
description: What it means for a problem to be in P or NP, what NP-complete and NP-hard add on top, and what the P=NP question actually asks.
roadmap: datastructures-and-algorithms
node: complexity-classes-p-np-and-np-complete
group: complexity-and-math-foundations
video:
  provider: youtube
  id: "EHp4FPyajKQ"
  start: 0
  end: null
creator:
  name: Up and Atom
  channel_url: https://www.youtube.com/@upandatom
duration_seconds: 960
level: intermediate
prerequisites:
  - asymptotic-notation-big-o-theta-and-omega
takeaways:
  - "P is the set of decision problems solvable in polynomial time; NP is the set of decision problems whose \"yes\" answers can be verified in polynomial time, even if no fast way to find that answer is known."
  - "Every problem in P is also in NP — if you can solve it quickly, you can certainly verify a solution quickly — but whether every problem in NP is also in P is the open P=NP question."
  - "NP-complete problems are the hardest problems in NP: every other NP problem can be transformed into one of them in polynomial time, so a fast solution to any single NP-complete problem would solve all of NP in polynomial time."
  - "NP-hard is a strictly broader label than NP-complete — it means \"at least as hard as the hardest NP problems,\" including problems that aren't even in NP because their solutions can't be verified quickly either."
  - "Co-NP is the set of problems whose \"no\" answers can be verified quickly, which is a genuinely different question from whether \"yes\" answers can be, and it's still open whether NP and co-NP are the same class."
check_questions:
  - q: What's the difference between a problem being "solvable in polynomial time" and "verifiable in polynomial time"?
    a: Solvable means there's an algorithm that finds a correct answer in polynomial time from scratch; verifiable means that given a proposed answer (a "certificate"), you can check whether it's correct in polynomial time, even if finding that answer in the first place might take far longer — NP is defined by the second, weaker property.
  - q: If someone proved P = NP tomorrow, what would that mean in practice?
    a: It would mean every problem whose solutions can be quickly checked also has a fast algorithm to find a solution outright, collapsing the distinction between "easy to verify" and "easy to solve" — with major practical consequences, since many of today's cryptographic systems rely on certain problems being hard to solve but easy to verify.
  - q: Why does proving a problem is NP-complete matter, if it doesn't tell you how to solve it?
    a: It tells you that no better-than-exponential algorithm is known for any problem in this enormous and well-studied class, so instead of spending more effort hunting for a fast exact algorithm, it's usually more productive to look for approximations, heuristics, or special-case restrictions that are tractable.
deeper:
  - label: "16. Complexity: P, NP, NP-completeness, Reductions"
    url: "https://www.youtube.com/watch?v=eHZifpgyH_4"
related_formulas: []
---


## Sorting problems by how hard they are

Big-O and Big-Theta classify a single algorithm's growth rate. Complexity classes do something different: they classify entire *problems*, independent of which algorithm you throw at them. Asking "is sorting hard?" is really asking "does every algorithm that solves sorting take at least this much time?" — and complexity theory gives you the vocabulary to make that question precise.

## P: solvable quickly

A decision problem (one with a yes/no answer) is in **P** if there's an algorithm that solves it in time polynomial in the size of the input — O(n), O(n²), O(n³), and so on, as opposed to O(2ⁿ) or O(n!). Sorting a list, finding the shortest path between two nodes in a graph (Dijkstra's algorithm, covered elsewhere in this roadmap), and checking whether a number is prime are all in P. "Polynomial" is the agreed-upon dividing line between "tractable" and "intractable" in complexity theory, even though a huge-degree polynomial like n¹⁰⁰ is no one's idea of fast in practice — the line is drawn there because polynomial-time algorithms compose well (a polynomial algorithm calling another polynomial algorithm is still polynomial) and because, empirically, almost every problem anyone has found a polynomial algorithm for has a *reasonably small-degree* one.

## NP: verifiable quickly

A decision problem is in **NP** ("nondeterministic polynomial time") if, given a proposed solution (a *certificate*), you can verify whether it's correct in polynomial time. The classic example is the Subset Sum problem: "does some subset of these numbers add up to exactly k?" Finding such a subset might require checking exponentially many combinations. But if someone *hands you* a candidate subset, adding it up and comparing to k takes linear time. That asymmetry, hard to find but easy to check, is the defining shape of NP.

Every problem in P is trivially in NP too: if you can solve it quickly, you can "verify" any proposed answer by just solving it yourself and comparing. The real question the field has been stuck on since the 1970s is the reverse direction: is every problem whose answers are easy to *check* also easy to *solve*? That's the P vs NP question, and despite enormous effort (and a $1 million Millennium Prize sitting on the answer), nobody has proven it either way. Most researchers suspect P ≠ NP — that verifying really is fundamentally easier than solving for at least some problems — but suspicion isn't proof.

## NP-complete: the hardest problems in NP

Not all NP problems are equally hard. **NP-complete** problems are the hardest ones in the class, defined by two properties: they're in NP, and every other problem in NP can be transformed ("reduced") into them using a polynomial-time transformation. That second property is the powerful one — it means an NP-complete problem is at least as hard as *every* problem in NP, because if you could solve it quickly, you could solve any NP problem quickly by first converting it into your NP-complete problem and then solving that.

The Boolean satisfiability problem (SAT) was the first problem proven NP-complete (the Cook-Levin theorem): given a logical formula, does some assignment of true/false to its variables make the whole thing true? Once one NP-complete problem exists, proving a new problem is NP-complete becomes a matter of reducing a *known* NP-complete problem to it. That's how thousands of problems have joined the list, among them the Traveling Salesman Problem, graph coloring, and the Knapsack problem (covered in this roadmap's dynamic-programming lesson). If anyone ever found a polynomial-time algorithm for *any single one* of them, it would prove P = NP and instantly solve every problem in NP in polynomial time, since they're all reducible to each other.

## NP-hard: dropping the "in NP" requirement

**NP-hard** problems are at least as hard as NP-complete problems — every NP problem reduces to them — but they drop the requirement of being in NP themselves. This matters because some problems are so hard that you can't even verify a proposed solution in polynomial time, let alone find one. The general Traveling Salesman Problem phrased as an optimization ("find the *shortest possible* route") rather than a decision ("is there a route shorter than k?") is NP-hard but not NP in the strict sense, since checking "is this really the shortest possible route" isn't itself a quick verification. Every NP-complete problem is NP-hard, but not every NP-hard problem is NP-complete — NP-hard is the broader, looser label.

## Co-NP: verifying "no" instead of "yes"

**Co-NP** is defined symmetrically to NP but for "no" answers: a problem is in co-NP if, whenever the answer is "no," there's a certificate that lets you verify that quickly. For Subset Sum, verifying a "yes" answer is easy (here's the subset, check the sum) but verifying a "no" answer — proving *no* subset works — isn't obviously easy, since ruling out every possibility seems to require the same exhaustive search as finding one in the first place. Whether NP and co-NP are actually the same class is itself an open problem, closely related to (but distinct from) P vs NP; most researchers suspect they're different, which would imply P ≠ NP as a consequence.

## Why this classification earns its keep

None of this tells you how to solve a hard problem faster. What it tells you is when to *stop looking* for an efficient exact algorithm and switch strategies — to an approximation algorithm that gets close to optimal in polynomial time, a heuristic that works well in practice without a worst-case guarantee, or a restriction to a special case of the problem that happens to be tractable. Recognizing "this is NP-complete" is itself a useful, actionable piece of information, because it tells you that decades of the smartest people in the field looking for a fast exact solution have come up empty — and that your time is better spent elsewhere.
