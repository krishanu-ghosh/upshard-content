---
title: "Compilation vs. Interpretation"
description: The spectrum from executing an AST directly to compiling all the way to machine code ahead of time — and where JIT compilation sits in between.
roadmap: computer-science
node: compilation-vs-interpretation
group: theory-of-computation-and-compilers
video:
  provider: youtube
  id: "d7KHAVaX_Rs"
  start: 0
  end: null
creator:
  name: Computerphile
  channel_url: https://www.youtube.com/@Computerphile
duration_seconds: 640
level: beginner
prerequisites:
  - lexing-and-parsing
takeaways:
  - "A pure interpreter walks the AST produced by parsing and executes it directly, re-doing that walk, and re-making the same decisions, every single time the code runs."
  - "A pure ahead-of-time (AOT) compiler translates the AST all the way down to native machine code before the program ever runs, so execution later involves no translation step at all."
  - "Compilation vs. interpretation is a spectrum of implementation strategies, not a property of a language — the same language can legally have both an interpreter and a compiler, and many languages have several of each."
  - "A JIT compiler starts by interpreting, watches which code actually runs often, and compiles only that hot code to machine code at runtime — trading some startup cost for speed that keeps improving as the program keeps running."
  - "The real tradeoff is startup time vs. peak throughput vs. portability: interpreters start instantly but run slower, AOT compilers are fast but need a separate build per target platform, and JIT compilers try to get both at the cost of extra runtime complexity and memory."
check_questions:
  - q: If a pure interpreter and a JIT compiler can both run the exact same Python program, what's actually different about the work each one does while it runs?
    a: The interpreter re-walks the AST (or re-executes similar bytecode instructions) every time it reaches that code, redoing the same work on every pass. The JIT compiler notices a piece of code is running repeatedly, compiles it to machine code once, and then runs that compiled version directly on later passes, skipping the repeated interpretation overhead.
  - q: Why might a short-lived command-line script favor a pure interpreter over a JIT compiler, even though the JIT would eventually run the code faster?
    a: A JIT compiler needs the program to run long enough, and the same code to execute often enough, to pay back the time spent profiling and compiling it. A script that runs once for a fraction of a second never reaches that break-even point, so all the JIT's warm-up cost is pure overhead with no payoff — a pure interpreter just starts running immediately instead.
  - q: Why is "Python is an interpreted language" a less precise statement than it sounds?
    a: Interpretation vs. compilation describes a specific implementation's strategy, not an inherent property of the language itself. CPython mostly interprets (via bytecode), but PyPy JIT-compiles Python, and tools exist that compile Python ahead of time — so the same language has implementations spanning the whole spectrum.
deeper:
  - label: "Just-in-time compilation - Wikipedia"
    url: "https://en.wikipedia.org/wiki/Just-in-time_compilation"
related_formulas: []
---

## Picking up where parsing left off

The previous lesson ended with an AST — a tree that captures the grammatical structure of a program, like `2 + 3 * 4` becoming a `+` node with `2` on one side and a `3 * 4` subtree on the other. That tree is a representation of *what the program means*, but it isn't yet something a CPU can run. This lesson is about the different strategies for turning that tree into actual execution, and they form a genuine spectrum rather than a binary choice.

## One end of the spectrum: walk the tree every time

The simplest strategy is a **tree-walking interpreter**: write a function that takes an AST node and executes it directly, recursing into child nodes as needed. For the `2 + 3 * 4` tree, that function sees a `+` node, recursively evaluates the left child (`2`, trivially), recursively evaluates the right child (a `*` node, which itself recurses into `3` and `4`), and adds the two results. No translation happens — the tree itself is the program, interpreted on the fly.

This is appealing because it's simple to build and starts running instantly: there's no separate build step, no waiting for a compile phase, just parse and immediately start walking. The cost shows up on repetition. If this expression sits inside a loop that runs a million times, the interpreter re-identifies that this is a `+` node, re-dispatches to the addition logic, and re-recurses into the same children a million separate times. It's redoing identical decision-making work on every single pass, because the interpreter never remembers that it already figured out what kind of node this is.

## The other end: translate everything, once, before running

An **ahead-of-time (AOT) compiler** goes the opposite direction: before the program runs at all, it walks the entire AST and emits native machine code — actual CPU instructions for the target processor — that, when executed, produces the same result the tree-walk would have. For `2 + 3 * 4`, a real compiler would likely even compute the constant result (`14`) at compile time and emit an instruction that just loads `14`, since the values never change — an optimization called constant folding that's only possible because the compiler can look at the *entire* program before any of it runs, something a line-by-line interpreter structurally cannot do.

Once compiled, running the program involves zero translation overhead — the CPU just executes instructions directly, which is why AOT-compiled languages (C, Rust, Go) tend to dominate benchmarks that measure raw execution speed. The cost is paid up front instead: a build step that takes real time, and machine code that's tied to a specific target (a program compiled for x86 won't run on ARM without being recompiled). That's a portability cost an interpreter doesn't have, since porting an interpreter means porting one program, not every program it runs.

## The middle: compile the parts that are worth compiling

A **JIT (just-in-time) compiler** is built to get benefits from both ends. It starts out behaving like an interpreter — running the AST, or more often a simpler intermediate form called bytecode (the subject of the next lesson), directly — while quietly keeping counters on which pieces of code actually execute. Once some function or loop crosses a "this is clearly hot" threshold, the JIT compiles *that specific code* to native machine code on the spot, during the running program's own execution, and subsequent calls jump straight to the compiled version instead of being reinterpreted.

This is how JavaScript engines like V8, the JVM's HotSpot, and PyPy get much closer to AOT-compiled speed while keeping an interpreter's "just run the file, no separate build step" experience. The price is complexity and memory: the runtime now has to carry both an interpreter and a compiler, profile code while it runs, and pay a genuine warm-up period during which code hasn't been compiled yet and runs at interpreter speed. That's exactly why short, quick scripts sometimes don't benefit from a JIT at all, and why benchmarks of JIT-compiled languages often look worse on tiny inputs and much better on long-running workloads.

## Why this is a spectrum, not a label on a language

None of this is actually a property of a programming language — it's a property of a specific implementation. Nothing about Python the language requires interpretation; CPython happens to interpret bytecode, PyPy JIT-compiles, and Cython and Nuitka compile Python ahead of time. The same is true in the other direction: C is "the compiled language" by reputation, but interpreters for C exist too. The real decision any language implementer faces is exactly the tradeoff this lesson described — startup latency, peak throughput, and portability — and different implementations of the same language are free to land in different places on that spectrum depending on what their users need most.
