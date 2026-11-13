---
title: "Turing Machines and Decidability"
description: A more powerful machine than a finite automaton — and a proof, using that machine, that some problems can never be solved by any program.
roadmap: computer-science
node: turing-machines-and-decidability
group: theory-of-computation-and-compilers
video:
  provider: youtube
  id: "macM_MtS_w4"
  start: 0
  end: null
creator:
  name: Computerphile
  channel_url: https://www.youtube.com/@Computerphile
duration_seconds: 374
level: beginner
prerequisites:
  - finite-automata-and-regular-languages
takeaways:
  - "A Turing machine extends a finite automaton with an infinite tape and a head that can move left or right and overwrite symbols, giving it genuine memory instead of just a current state."
  - "That extra memory is what makes a Turing machine strictly more powerful than a finite automaton — it can recognize languages, like balanced parentheses, that no finite automaton can."
  - "The Church-Turing thesis holds that a Turing machine can compute anything any realistic computer, including the one you're reading this on, can compute — it's the ceiling on computation, not just one model among many."
  - "A problem is decidable if some Turing machine is guaranteed to halt and give the correct yes/no answer on every possible input."
  - "The halting problem — deciding whether an arbitrary program halts on an arbitrary input — is proven undecidable: no algorithm can solve it correctly for every possible program, ever, no matter how clever or how much time or memory it's given."
check_questions:
  - q: A finite automaton can't match balanced parentheses because it has no way to count. Why does a Turing machine's tape fix that specific problem?
    a: The tape gives the machine a place to write symbols down and read them back later — for example, it can write a mark for each open paren it sees and erase one for each close paren, using the tape itself as an unbounded counter, which a finite automaton's fixed set of states can never provide.
  - q: What exactly does it mean for a problem to be "undecidable," as opposed to just "really hard" or "not yet solved"?
    a: Undecidable means it's been mathematically proven that no algorithm, run on any computer for any amount of time, can correctly answer the question for every possible input — it's not a matter of needing more computing power or a cleverer programmer; a correct general solution has been shown not to exist.
  - q: Why can't you solve the halting problem in general by just running the program and seeing if it stops?
    a: If it stops, you'll eventually see that — but if it doesn't stop, "just running it" never finishes either, so you can wait forever without ever being sure the answer is "no." You need a method that gives a correct yes-or-no answer in finite time for every program, and the proof shows no such method can exist.
deeper:
  - label: "Turing Machines Explained - Computerphile"
    url: "https://www.youtube.com/watch?v=dNRDvLACg5Q"
related_formulas: []
---

## Giving the machine somewhere to write things down

The previous lesson's finite automaton could only ever remember which of its finitely many states it was in. A **Turing machine** fixes that by adding two things: an infinite **tape**, divided into cells that each hold one symbol, and a **head** positioned over one cell at a time. At each step, the machine reads the symbol under the head, and based on that symbol and its current state, it writes a new symbol into that cell, moves the head one cell left or right, and transitions to a new state. It keeps doing this until it reaches a designated **halt** state (or runs forever).

That sounds like a small upgrade, but it's the difference between a machine with no memory and a machine with unbounded memory. The tape can be used as scratch space — a counter, a stack, a second copy of the input to compare against — because the machine can come back and re-read, and overwrite, anything it wrote earlier. A finite automaton can't do any of that; once a symbol has been consumed, it's gone.

## Why this particular machine matters so much

A Turing machine can now solve problems a finite automaton provably cannot, like checking whether a string of parentheses is balanced: walk right, and every time you see `(`, write a mark further down the tape; every time you see `)`, erase one mark; if you run out of marks to erase, or finish with marks left over, reject. That's a trivial program for a Turing machine and an impossible one for a finite automaton, because it requires counting without a predetermined bound.

What makes the Turing machine the central object of the whole field, though, is the **Church-Turing thesis**: the claim — overwhelmingly supported by decades of trying to find a counterexample — that anything computable by any realistic model of computation (your laptop, a quantum computer, a massive datacenter) is also computable by some Turing machine. It isn't one model among several equally powerful ones; it's believed to be a ceiling on computation itself. That's why "is this Turing-computable?" and "can any computer, in principle, ever compute this?" are treated as the same question.

## Decidability: when a yes/no question has a guaranteed answer

A problem is **decidable** if there's a Turing machine that, given any valid input, always halts and gives the correct answer — "yes" or "no," with no third option of running forever. Plenty of problems are decidable this way: "is this number prime?", "does this regular expression match this string?", "is this list sorted?" All of these have algorithms that are guaranteed to finish and get it right.

The halting problem asks something different: given the description of an arbitrary program and an input, will that program eventually halt, or run forever? It would be enormously useful to have a tool that always answers this correctly — imagine a compiler warning that definitively flagged every infinite loop. Turing proved in 1936 that no such tool can exist, for any program whatsoever.

## Why no algorithm can ever do this

The proof is a diagonalization argument, and it's worth following once because the technique recurs throughout computer science. Suppose, for contradiction, a program `H(P, input)` existed that always correctly decided whether program `P` halts on `input`. Build a new program `D(P)` that runs `H(P, P)` — asking whether `P` halts when given its own source code as input — and then does the opposite of whatever `H` predicted: if `H` says "`P` halts," `D` loops forever; if `H` says "`P` loops forever," `D` halts immediately.

Now ask: what does `D(D)` do? If `H` predicts `D(D)` halts, then by `D`'s own construction it loops forever instead — contradicting the prediction. If `H` predicts it loops forever, `D` halts immediately instead — also a contradiction. Either way, `H`'s prediction about `D(D)` is wrong. Since `H` was assumed to always be correct, no such `H` can exist. The halting problem isn't merely unsolved — it's been proven, by direct contradiction, to be unsolvable by any algorithm at all, which is why real tools (compilers, static analyzers) only ever approximate it, correctly flagging some infinite loops while necessarily missing or misflagging others.
