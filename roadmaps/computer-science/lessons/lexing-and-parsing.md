---
title: "Lexing and Parsing"
description: How raw source text becomes tokens, and how those tokens become a tree — the two-stage pipeline every compiler and interpreter starts with.
roadmap: computer-science
node: lexing-and-parsing
group: theory-of-computation-and-compilers
video:
  provider: youtube
  id: "OIKL6wFjFOo"
  start: 0
  end: null
creator:
  name: Neso Academy
  channel_url: https://www.youtube.com/@nesoacademy
duration_seconds: 852
level: beginner
prerequisites:
  - finite-automata-and-regular-languages
takeaways:
  - "Lexing (scanning) turns a flat stream of characters into a flat stream of tokens, each tagged with a type like NUMBER, PLUS, or IDENTIFIER — it throws away irrelevant detail like whitespace along the way."
  - "A lexer's per-token rules are essentially regular expressions, which is why the finite-automaton theory from the previous lesson is exactly what a real lexer is built on."
  - "Parsing takes that flat token stream and imposes structure on it, building a tree (a parse tree or abstract syntax tree, AST) that reflects how the tokens group according to the language's grammar."
  - "Parsing generally needs more expressive power than lexing does, because grammatical structure like nested expressions and matching brackets isn't a regular language — it needs a context-free grammar."
  - "Operator precedence (why 2 + 3 * 4 is 14, not 20) isn't something the lexer knows or cares about; it's entirely a property of how the parser's grammar is structured."
check_questions:
  - q: Why can a lexer get away with using finite-automaton-style pattern matching, when the previous lesson showed finite automata can't handle nested structure?
    a: A lexer's job is only to recognize individual tokens — a number, an identifier, an operator — each of which is a flat, non-nested pattern with no unbounded counting involved. The nested structure of the whole program is a separate problem that gets handled one stage later, by the parser.
  - q: If both the lexer and the parser produce a kind of structured output, what's the actual difference between a token stream and a parse tree?
    a: A token stream is flat — just a sequence with no relationships between elements beyond their order. A parse tree is hierarchical — it encodes which tokens belong to which sub-expression, which is exactly the grouping information a flat list cannot represent.
  - q: Where does "multiplication binds tighter than addition" actually live in a compiler's pipeline — in the lexer, or the parser?
    a: It lives in the parser's grammar. The lexer only identifies that * and + are each operator tokens; it has no concept of which one should group its operands first. That grouping decision is made when the parser decides how to build the tree, which is exactly what makes 2 + 3 * 4 parse as 2 + (3 * 4).
deeper:
  - label: "Introduction to Lexical Analyzer - Neso Academy"
    url: "https://www.youtube.com/watch?v=4nx8LEGy9kI"
related_formulas: []
---

## Stage one: turning characters into tokens

Source code arrives as a flat sequence of characters — nothing more. Before a compiler or interpreter can do anything meaningful with `2 + 3 * 4`, it needs to group those characters into meaningful chunks: the characters `2`, the character `+`, the characters `3`, and so on aren't useful one at a time. That grouping step is **lexing**, also called **scanning** or **tokenization**.

A lexer scans left to right and, at each point, matches the longest prefix of the remaining input against a set of patterns — one pattern per **token type**. Digits in a row become a `NUMBER` token; a run of letters becomes an `IDENTIFIER` (or a `KEYWORD`, if it matches a reserved word like `if`); a single `+` becomes a `PLUS` token. Whitespace is typically matched and then discarded, since it carries no meaning beyond separating other tokens. For `2 + 3 * 4`, the lexer produces: `NUMBER(2)`, `PLUS`, `NUMBER(3)`, `TIMES`, `NUMBER(4)` — a flat list, five tokens long, with the original spacing gone.

This is precisely where the previous lesson's finite automata come back. Each token type's pattern — "one or more digits," "a letter followed by letters or digits" — is a regular language, and real lexer generators (like `lex` or `flex`) compile these patterns into exactly the kind of state-machine the previous lesson described, then run all of them over the input simultaneously, picking whichever matches the longest run of characters at each position. A lexer is, quite literally, a finite automaton doing its job.

## Stage two: turning tokens into structure

A flat list of tokens doesn't tell you how they relate to each other. `NUMBER(3)` and `TIMES` are adjacent in the list, but so are `PLUS` and `NUMBER(3)` — the list alone doesn't say that `3 * 4` should be treated as a unit before adding `2` to it. Recovering that structure is **parsing**: taking the token stream and building a tree — a **parse tree**, or its more streamlined cousin the **abstract syntax tree (AST)** — whose shape reflects the grammatical structure of the input.

For `2 + 3 * 4`, a parser that knows multiplication binds tighter than addition builds a tree with `+` at the root, `2` as its left child, and an entire `*` subtree — with `3` and `4` as its children — as its right child. Reading that tree back out, evaluating the deepest nodes first, gives `3 * 4 = 12`, then `2 + 12 = 14`. The *same five tokens*, parsed with the opposite precedence assumption, would instead produce a tree with `+` nested inside `*`, evaluating to `(2 + 3) * 4 = 20`. Nothing about the token stream itself decides between these — the grammar the parser is built from is what decides it, which is why language designers write precedence tables and why `3 + 4 * 5` isn't ambiguous in any language that defines its grammar properly.

## Why parsing generally needs more than a regular language

This is also where the ceiling from the previous lesson becomes unavoidable. Consider matching parentheses in something like `(2 + (3 * 4))` — a parser has to track "how many open parens are currently unmatched" to know when a close paren is valid and when expression grouping ends, and that's exactly the kind of unbounded counting a finite automaton (and hence a straightforward lexer) cannot do. Parsing is instead built on **context-free grammars**, a strictly more expressive notation that can describe rules like "an expression is either a number, or an expression plus an expression, or an open paren, an expression, and a close paren" — a rule that refers to itself, letting it describe structures nested arbitrarily deep.

A real parser applies these grammar rules recursively: to parse an expression, it may need to parse a sub-expression first, which may itself contain another sub-expression, and so on, which is why this family of techniques is called recursive descent, and why the data structure it naturally produces is a tree rather than a flat list.

## Why the split exists at all

It would be possible, in principle, to skip straight from characters to a tree in one pass. Compilers don't, because splitting the work keeps each stage simple: the lexer only ever has to answer "what kind of token is this," using fast, well-understood automaton techniques, while the parser only ever has to answer "how do these already-identified tokens fit together," using grammar rules that don't need to know anything about individual characters, whitespace, or comments. That separation of concerns is also why the next lesson's distinction between interpreting and compiling a program only matters *after* this pipeline has already produced a tree to work with — lexing and parsing are the front door every language implementation walks through first.
