---
title: "Instruction Sets and Assembly Basics"
description: What an instruction set architecture is, RISC vs CISC at a conceptual level, and a short assembly snippet walked through line by line.
roadmap: computer-science
node: instruction-sets-and-assembly-basics
group: computer-architecture
video:
  provider: youtube
  id: "zltgXvg6r3k"
  start: 0
  end: null
creator:
  name: CrashCourse
  channel_url: https://www.youtube.com/@crashcourse
duration_seconds: 614
level: beginner
prerequisites:
  - how-a-cpu-executes-instructions
takeaways:
  - "An instruction set architecture (ISA) is the contract between hardware and software: the exact list of instructions a CPU understands, their binary encodings, and the registers and addressing modes available, independent of how any particular chip implements them internally."
  - "Assembly language is a human-readable, near-1-to-1 text representation of machine code instructions - one assembly line generally corresponds to one instruction, unlike a high-level language where one line can expand into many."
  - "RISC (reduced instruction set) designs favor a small number of simple, fixed-length, single-cycle-ish instructions, pushing complexity into software and compilers, while CISC (complex instruction set) designs favor fewer lines of assembly per task via richer, variable-length instructions that can do more per instruction."
  - "The RISC vs CISC split is a design philosophy about where complexity lives, not a strict boundary - modern x86 CISC chips internally translate complex instructions into simpler RISC-like micro-operations before executing them."
  - "Because an ISA is a fixed contract, the same compiled binary can run on any CPU that implements that ISA, which is why x86 software doesn't run unmodified on an ARM chip - they're different instruction sets entirely."
check_questions:
  - q: "Why can't a program compiled for one instruction set architecture just run on a CPU built for a different ISA?"
    a: "The compiled binary is a sequence of instruction encodings specific to the ISA it was built for - the same bit pattern means a completely different operation (or nothing valid at all) under a different ISA, since each architecture defines its own opcodes, register set, and encoding rules. Running it anywhere else requires either recompiling the source for that ISA or translating/emulating the instructions, both of which cost something RISC vs CISC doesn't fix."
  - q: "If RISC instructions do less work each, why would a RISC CPU ever be faster overall than a CISC CPU doing the 'same' task?"
    a: "Simpler, more uniform instructions are easier to decode quickly and easier to pipeline efficiently, since every instruction takes roughly the same shape and amount of work to process - so a RISC CPU can often execute more instructions per cycle even though it needs more of them in total; CISC's advantage is fewer instructions per task, not necessarily faster execution of each one."
  - q: "What does it mean that modern x86 chips translate CISC instructions into 'micro-ops' before running them?"
    a: "It means the chip doesn't execute the complex x86 instruction directly in hardware as one atomic unit - internally, a decoder breaks it down into one or more simpler, RISC-like operations that the chip's actual execution pipeline runs, letting x86 keep its established instruction encoding and software compatibility on the outside while gaining much of the pipelining and scheduling efficiency of a RISC-style core on the inside."
deeper:
  - label: "RISC vs CISC - GeeksforGeeks"
    url: "https://www.geeksforgeeks.org/computer-organization-architecture/computer-organization-risc-and-cisc/"
related_formulas: []
---

## The instruction set as a contract

An **instruction set architecture (ISA)** is the complete specification of what a CPU can be told to do: the list of valid instructions, how each one is encoded as bits, what registers exist and how many there are, and what addressing modes are available for referring to memory. It says nothing about *how* the chip internally executes those instructions - that's left to the specific implementation (the microarchitecture), which is why two CPUs can both implement the x86-64 ISA and run the exact same compiled programs while having very different internal pipeline designs, cache sizes, and performance characteristics.

This is why the ISA matters so much in practice: it's the stable boundary between hardware and software. A compiler targets an ISA, not a specific chip, and any CPU implementing that ISA can run the resulting binary. It's also why a binary compiled for x86-64 won't run on an ARM-based machine (like an Apple Silicon Mac) without translation or emulation - ARM and x86-64 are different ISAs with different instruction encodings, register sets, and conventions, even though both are perfectly capable of running the same algorithms once compiled separately for each.

## RISC and CISC: two philosophies about where complexity goes

**CISC** (Complex Instruction Set Computer) architectures, like the original x86 design, pack a lot of functionality into single instructions - one instruction might load a value from memory, perform an operation on it, and store the result, all in one encoded step, with variable-length instructions sized to fit whatever that instruction needs to express. The appeal is that fewer, denser instructions are needed to express a given task, which mattered a great deal when memory was scarce and expensive.

**RISC** (Reduced Instruction Set Computer) architectures, like ARM and RISC-V, instead favor a smaller set of simple, uniform instructions - typically fixed-length, each doing one basic thing (one load, or one arithmetic operation, but rarely both combined), with memory access and computation kept as separate instructions. This uniformity makes each instruction easier and faster to decode, and far easier to pipeline efficiently, since the hardware doesn't have to handle wildly different instruction shapes and costs.

Neither approach is simply "better" - it's a trade-off between instruction density and hardware simplicity. Interestingly, the distinction has blurred over time: modern x86 chips are CISC on the outside (for software compatibility) but internally decode each complex instruction into simpler, RISC-like micro-operations that the actual execution hardware runs, borrowing RISC's pipelining advantages without breaking decades of x86 software compatibility.

## A short assembly snippet, line by line

Assembly language is a thin, human-readable layer over machine code: each line generally corresponds to one actual instruction the CPU will execute, using short mnemonics instead of raw binary. Here's a small RISC-style snippet that adds two numbers and stores the result, using a made-up but representative instruction set:

```
    LOAD  R1, [A]     ; R1 = value stored at memory location A
    LOAD  R2, [B]     ; R2 = value stored at memory location B
    ADD   R3, R1, R2  ; R3 = R1 + R2
    STORE [C], R3     ; store R3's value into memory location C
```

- **`LOAD R1, [A]`** - fetches the value sitting at memory address `A` and copies it into register R1. The brackets around `A` signal "the value *at* this address," not the address itself.
- **`LOAD R2, [B]`** - same idea, pulling the value at address `B` into register R2. Note both operands have to be loaded into registers first; a RISC design doesn't allow arithmetic directly on memory operands.
- **`ADD R3, R1, R2`** - the ALU adds the contents of R1 and R2 and writes the sum into R3. This instruction only ever touches registers, which is exactly the kind of uniform, single-purpose instruction RISC designs favor.
- **`STORE [C], R3`** - takes the value now sitting in R3 and writes it out to memory address `C`, making the result visible outside the CPU's registers.

A CISC version of the same task might compress this into fewer lines - for example, one instruction that adds a memory operand directly to a register and writes straight back to memory - at the cost of that single instruction being more complex to decode and execute than any one RISC instruction above. Either way, the underlying work is identical: two values are read, added, and the result is stored. The ISA only decides how finely that work gets sliced into instructions, and how much of the slicing is the compiler's job versus the hardware's.
