---
title: "Intermediate Representations and Bytecode"
description: Why compilers and runtimes translate to bytecode instead of going straight from AST to machine code, using Python's .pyc and JVM bytecode as examples.
roadmap: computer-science
node: intermediate-representations-and-bytecode
group: theory-of-computation-and-compilers
video:
  provider: youtube
  id: "GNPKBICTF2w"
  start: 0
  end: null
creator:
  name: EuroPython Conference
  channel_url: https://www.youtube.com/@EuroPythonConference
duration_seconds: 1357
level: beginner
prerequisites:
  - compilation-vs-interpretation
takeaways:
  - "An intermediate representation (IR) is a format a compiler translates source into partway to its final target — flatter and simpler than an AST, but not yet tied to any specific CPU."
  - "Bytecode is a common style of IR: a sequence of simple, numbered instructions for a stack-based or register-based virtual machine that doesn't exist as real hardware."
  - "Python compiles source to bytecode and caches it in a .pyc file so that re-running the same unchanged file skips lexing and parsing on later runs, not just skipping a round trip to a CPU."
  - "The JVM compiles Java (and Kotlin, Scala, Clojure...) source to one shared bytecode format, which is what lets a single compiled .class file run unmodified on any machine with a JVM, regardless of the underlying CPU architecture."
  - "Working on an IR instead of either an AST or raw machine code makes optimization passes reusable across source languages and reusable across target CPUs, instead of needing to be written once for every language-times-CPU combination."
check_questions:
  - q: An AST already captures the program's structure. Why isn't that enough to hand straight to a CPU or an optimizer?
    a: An AST mirrors the source language's own grammar — nested expressions, language-specific constructs — which is convenient for parsing but awkward to reason about uniformly or translate to hardware. Bytecode flattens that structure into a linear sequence of simple instructions that are both easier to optimize mechanically and much closer to what a real (or virtual) processor actually executes.
  - q: Two completely different languages, say Kotlin and Clojure, can both run on the JVM. What does that actually require being shared between them?
    a: Both languages' compilers have to translate their very different source syntax down into the same JVM bytecode format. Once that translation happens, the JVM itself never needs to know or care which source language produced the bytecode — portability and interoperability both come from that one shared target.
  - q: If you edit a .py file and immediately rerun it, Python usually recompiles it — but if you don't touch it, the next run can skip straight to execution. What is Python actually checking to decide which happens?
    a: CPython compares the source file's modification time (and sometimes a hash) against metadata stored in the cached .pyc file. If the source hasn't changed since that bytecode was generated, it reuses the cached bytecode and skips lexing and parsing entirely; if the source changed, the mismatch forces a fresh compile.
deeper:
  - label: "JVM bytecode - Wikipedia"
    url: "https://en.wikipedia.org/wiki/Java_bytecode"
related_formulas: []
---

## The step compilers take between the AST and the metal

The previous two lessons built up a pipeline: characters to tokens, tokens to an AST, and then the AST either walked directly (interpretation) or translated all the way to machine code (compilation). In practice, almost no real compiler or language runtime jumps straight from AST to machine code. They pass through one or more **intermediate representations (IRs)** first — and the most common style of IR, especially for interpreted and JIT-compiled languages, is **bytecode**.

Bytecode is a sequence of small, numbered instructions meant for a **virtual machine** — a hypothetical processor that doesn't exist as physical hardware, but that's simple and uniform enough to interpret quickly or compile from easily. Where an AST for `2 + 3 * 4` is a tree with nested nodes, the equivalent bytecode is a flat list: something like `LOAD_CONST 2`, `LOAD_CONST 3`, `LOAD_CONST 4`, `BINARY_MULTIPLY`, `BINARY_ADD`. Each instruction does one small, well-defined thing to a stack of values, and there's no nesting left to interpret — just march down the list, instruction by instruction.

## Why flatten the tree into a list at all

A tree is awkward to execute mechanically and awkward to optimize mechanically, because every kind of AST node needs its own special-case handling, and that handling is specific to one particular source language's grammar. A flat instruction list, by contrast, can be processed with simple, uniform passes: an optimizer can scan straight through looking for patterns like "two `LOAD_CONST` instructions followed by an arithmetic op" and replace all three with one `LOAD_CONST` holding the precomputed result — constant folding, performed mechanically on a list instead of needing tree-recursion logic specific to this one language's AST shape. The same optimizer, written once against the bytecode format, works for every language that compiles to that format, and the same bytecode interpreter, written once, works for every program in that format regardless of which language produced it.

## Python's .pyc: caching the translation, not just skipping the CPU

When you run a `.py` file, CPython doesn't interpret the raw source text line by line. It first lexes and parses it into an AST (as in the earlier lessons), compiles that AST down into Python bytecode, and only then runs that bytecode on the CPython virtual machine. Because lexing, parsing, and compiling all take real time, CPython caches the resulting bytecode in a `.pyc` file (inside a `__pycache__` directory for imported modules). The next time that exact, unchanged source file is imported, CPython checks the source's modification time against what's recorded in the cache and, if nothing changed, skips straight to loading the cached bytecode — lexing and parsing never happen again. Change one character in the source, though, and the timestamp mismatch forces a full recompile, because the cached bytecode no longer corresponds to the current source.

## JVM bytecode: one target, many source languages

The JVM takes the same idea and uses it for something more ambitious than caching: cross-language, cross-platform portability. `javac` compiles Java source into `.class` files containing JVM bytecode, but Java is far from the only language that targets this format — Kotlin, Scala, Clojure, and Groovy all compile to the identical bytecode format. None of those compilers need to know anything about the physical CPU the program will eventually run on, and the JVM itself doesn't need to know or care which source language, or which compiler, produced the bytecode it's executing. That's the famous "compile once, run anywhere" promise: the same `.class` file runs unmodified on an x86 laptop, an ARM phone, or a mainframe, because each platform supplies its own JVM that knows how to execute that one shared instruction set — either by interpreting it directly or, as covered in the previous lesson, JIT-compiling the hot parts to that specific machine's native code.

## What this design actually buys you

Stack this up against skipping the IR entirely: without a shared bytecode target, every language's compiler would need a separate, from-scratch backend for every CPU architecture it wants to support — a Kotlin-to-x86 backend, a Kotlin-to-ARM backend, and so on, with optimization logic duplicated in each one. With a shared IR in the middle, that becomes two much smaller problems instead of one enormous one: every source language needs exactly one compiler targeting the IR, and every target platform needs exactly one engine (interpreter and/or JIT) that runs the IR. Optimizations written against the IR benefit every source language at once; support for a new source language, or a new target platform, only requires plugging into that one shared interface rather than rebuilding every combination from scratch.
