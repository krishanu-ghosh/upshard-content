---
title: "Finite Automata and Regular Languages"
description: What a finite automaton is, what makes a language "regular," and why every regex engine you use is secretly running one.
roadmap: computer-science
node: finite-automata-and-regular-languages
group: theory-of-computation-and-compilers
video:
  provider: youtube
  id: "k3AP20v1yBw"
  start: 0
  end: null
creator:
  name: Neso Academy
  channel_url: https://www.youtube.com/@nesoacademy
duration_seconds: 756
level: beginner
prerequisites: []
takeaways:
  - "A finite automaton is a machine with a fixed, finite set of states, a start state, one or more accept states, and transitions that move it between states one input symbol at a time."
  - "A string is accepted exactly when feeding its symbols into the automaton, one by one, ends in an accept state after the input runs out."
  - "A regular language is defined as any language some finite automaton accepts — the automaton is the formal definition, not just an implementation detail."
  - "Every regular expression can be mechanically converted into an equivalent finite automaton, and every finite automaton into an equivalent regular expression: they describe exactly the same class of languages."
  - "Finite automata have no memory beyond which state they are in, which is precisely why they cannot count or match arbitrarily nested structure — that limit is what the next lesson's more powerful models exist to fix."
check_questions:
  - q: Why can't a finite automaton with, say, 5 states keep track of "how many times has this happened so far" once the count can grow without bound?
    a: The automaton's entire memory is which single state it's currently in, and it only has a fixed, finite number of states — so it can distinguish at most that many situations. An unbounded count needs infinitely many distinguishable situations, which a finite set of states structurally cannot provide.
  - q: If a regular expression and a finite automaton can both define the exact same language, why do programming languages give you regex syntax instead of making you draw state diagrams?
    a: They're mathematically interchangeable, but regex is a far more compact and readable notation for a human to write, while the automaton is what the regex engine actually compiles that notation into (conceptually) before using it to scan text.
  - q: A DFA for "strings over {0,1} containing an even number of 1s" only needs 2 states. Why 2, and not fewer or more?
    a: The only thing that ever matters for deciding acceptance is the parity of 1s seen so far, which has exactly two possible values (even or odd) — so 2 states is both necessary and sufficient to track every distinction the language actually depends on.
deeper:
  - label: "Regular Expression Matching Can Be Simple and Fast (Russ Cox)"
    url: "https://swtch.com/~rsc/regexp/regexp1.html"
related_formulas: []
---

## A machine that can only remember where it is

A finite automaton is one of the simplest possible models of computation, and that simplicity is the point. It consists of a finite set of **states**, one of which is the **start state**, some subset marked as **accept states**, and a **transition function** that says, for every state and every possible input symbol, which state to move to next. That's the entire machine. It has no scratch paper, no counter, no stack — the only thing it "remembers" at any moment is which single state it's currently sitting in.

To run it, you feed in a string one symbol at a time. Starting from the start state, each symbol triggers a transition to a new state. Once the string is exhausted, the automaton **accepts** if it landed on an accept state, and **rejects** otherwise. That's the whole algorithm: start, consume, consume, consume, check where you ended up.

## A worked example

Suppose you want to accept exactly the binary strings with an even number of 1s (including the empty string, which has zero 1s — even). You need only two states: `EVEN` (the start state and the only accept state) and `ODD`. On a `0`, stay put. On a `1`, flip between them. Feed in `1101`: start in `EVEN`, the first `1` moves you to `ODD`, the second `1` moves you back to `EVEN`, the `0` keeps you in `EVEN`, the final `1` moves you to `ODD`. You end in `ODD`, which isn't an accept state, so `1101` is rejected — correctly, since it has three 1s.

Notice what made two states enough: the only fact about the string-so-far that ever affects the final answer is the parity of 1s seen. The automaton doesn't need to remember the actual count, or the order of 0s and 1s, because none of that changes the outcome. Designing an automaton for a language is really an exercise in figuring out the smallest set of distinctions the problem requires you to track — and discovering that some problems require infinitely many such distinctions, which no finite automaton can supply.

## What "regular" actually means

A **regular language** is defined, formally, as any language that some finite automaton accepts. That's a slightly unusual kind of definition — it's not "a language with such-and-such syntactic shape," it's "a language for which this specific kind of machine exists." The payoff is a theorem that connects this machine-based definition to the notation you actually use every day: for every regular expression, there's an equivalent finite automaton that accepts exactly the strings it matches, and conversely, every finite automaton's language can be written as a regular expression. They're two notations for the same underlying class of languages — one built from machines with states and transitions, the other built from concatenation, alternation (`|`), and repetition (`*`).

This is why regex engines exist in the form they do. When you write `[a-z]+@[a-z]+\.com`, a regex engine doesn't search the string character by character trying every possibility from scratch. Conceptually, it compiles your pattern into something equivalent to a finite automaton, then runs that automaton over the input exactly the way described above: one symbol, one transition at a time, in a single pass. That's also why regex matching can be made extremely fast — linear in the length of the input — when implemented this way, in contrast to backtracking-based implementations that simulate a different, more expensive search strategy.

## Where the limit shows up

The flip side of this efficiency is a hard ceiling on expressive power. Because a finite automaton's only memory is its current state, it cannot match balanced parentheses, nested HTML tags, or anything requiring you to count an unbounded quantity and remember the count. Try to write a regex that matches "any number of open parens followed by exactly that many close parens" and you'll find it's impossible — not just hard, but provably impossible for any finite automaton, no matter how many states you throw at it. That gap between "patterns in flat text" and "nested, recursive structure" is exactly the boundary this lesson exists to mark. It's the reason the next lessons introduce machines with actual memory: first a model powerful enough to compute anything a modern machine can, and separately, a more structured notation — context-free grammars — strong enough to describe nested syntax, which is exactly what a programming language's parser needs.
