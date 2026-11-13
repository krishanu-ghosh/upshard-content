---
title: "How a CPU Executes Instructions"
description: The fetch-decode-execute cycle, registers versus RAM, the program counter, and a worked trace of a CPU running a few instructions.
roadmap: computer-science
node: how-a-cpu-executes-instructions
group: computer-architecture
video:
  provider: youtube
  id: "FZGugFqdr60"
  start: 0
  end: null
creator:
  name: CrashCourse
  channel_url: https://www.youtube.com/@crashcourse
duration_seconds: 675
level: beginner
prerequisites: []
takeaways:
  - "A CPU runs every program by repeating one loop, the fetch-decode-execute cycle, billions of times per second, never doing anything fundamentally more complex than that loop."
  - "Registers are a tiny number of storage slots built directly into the CPU, fast enough to be read or written in a single clock cycle, while RAM is far larger but sits across a bus the CPU has to wait on."
  - "The program counter is a register that holds the memory address of the next instruction to run, and it is what makes 'next' a well-defined idea instead of something a program has to track itself."
  - "Decoding an instruction means splitting its bits into an operation and its operands, which is what tells the ALU and control unit exactly what hardware to activate during execute."
  - "Because every instruction must be fetched from memory before it can run, the speed of that fetch step is a major reason the memory hierarchy and caching exist at all."
check_questions:
  - q: "Why can't a CPU just execute an instruction directly out of RAM, as opposed to the instruction register?"
    a: "An instruction's bits have to be held still inside the CPU while the control unit decodes them and routes signals to the ALU and registers over several sub-steps; RAM is too slow to be read on every one of those sub-steps, so the CPU fetches the instruction once into a dedicated register and works from that stable copy."
  - q: "If the program counter is corrupted and made to point at the wrong address, what actually goes wrong?"
    a: "The CPU doesn't know or care that the address is 'wrong' - it will fetch whatever bit pattern lives at that address and decode it as an instruction anyway, which usually means executing garbage or someone else's data as if it were code, often crashing the program or producing nonsense results."
  - q: "Why does the CPU increment the program counter during the fetch step rather than waiting until after the instruction finishes executing?"
    a: "Incrementing immediately after fetch means the PC always points at the next sequential instruction regardless of how long execute takes, and it also means a jump or branch instruction can simply overwrite the PC during its own execute step to redirect control flow, rather than fighting with a pending auto-increment."
deeper:
  - label: "Inside the CPU - Computerphile"
    url: "https://www.youtube.com/watch?v=IAkj32VPcUE"
related_formulas: []
---

## One loop, repeated billions of times a second

Everything a CPU does, from rendering a video frame to running a spreadsheet formula, reduces to the same three-step loop running over and over: **fetch**, **decode**, **execute**. There is no special hardware mode for "running a loop" or "calling a function" - those are just patterns of ordinary instructions executed one at a time, in this same cycle, millions or billions of times per second depending on the CPU's clock speed.

**Fetch** - the CPU looks at the **program counter (PC)**, a special register holding the memory address of the next instruction, and reads the bits stored at that address into the **instruction register**. It then increments the PC so it points at the next instruction in sequence, before the current one has even been decoded.

**Decode** - the control unit examines the bit pattern just fetched and figures out what it means: which operation to perform (add, load, store, jump, compare, ...) and which registers or memory addresses are the operands. Different bit fields within the instruction map to an opcode and to operand addresses, the way a sentence's grammar tells you which word is the verb and which are its objects.

**Execute** - the control unit activates the actual hardware the decoded instruction calls for. An arithmetic instruction drives the ALU; a load or store instruction drives the path to memory; a branch instruction may overwrite the program counter itself, redirecting what gets fetched next. Then the cycle repeats.

## Registers versus RAM: why both exist

A CPU could, in principle, do all of its work directly against RAM. It doesn't, because RAM is physically far away from the CPU's logic (relatively speaking - more wire length, more electrical distance) and built for density rather than speed. Reading from RAM takes many clock cycles.

**Registers** are the opposite trade-off: a CPU typically has only a few dozen general-purpose registers, each holding a single word (commonly 32 or 64 bits), but they sit directly inside the CPU's own silicon and can be read or written in a single clock cycle. The ALU only ever operates on values that are already in registers - to add two numbers sitting in memory, the CPU first has to load them into registers, add them there, and optionally store the result back.

This is why "registers vs. RAM" isn't a question of which is better - it's a deliberate hierarchy: a tiny amount of extremely fast storage next to the compute logic, backed by a much larger but slower pool for everything that doesn't fit. The next lesson digs into how this idea extends further, with caches sitting between the two.

## A worked trace

Imagine a tiny, simplified instruction set and this three-instruction program sitting in memory starting at address 100, with the PC currently holding `100`:

```
100: LOAD  R1, [500]   ; R1 = value at memory address 500
101: ADD   R1, R2      ; R1 = R1 + R2
102: STORE [500], R1   ; value at memory address 500 = R1
```

Suppose memory address 500 holds `7`, and register R2 already holds `3`.

**Cycle 1** - Fetch reads the bits at address 100 (`LOAD R1, [500]`) into the instruction register and sets PC to 101. Decode identifies this as a load, with destination register R1 and source address 500. Execute reads `7` from address 500 and writes it into R1. R1 now holds `7`.

**Cycle 2** - Fetch reads address 101 (`ADD R1, R2`) and sets PC to 102. Decode identifies an add with operands R1 and R2. Execute drives the ALU to compute `R1 + R2 = 7 + 3 = 10` and writes the result back into R1. R1 now holds `10`.

**Cycle 3** - Fetch reads address 102 (`STORE [500], R1`) and sets PC to 103. Decode identifies a store targeting address 500 from R1. Execute writes R1's value, `10`, to memory address 500, overwriting the `7` that was there.

After three trips around the cycle, the program has read a value from memory, added `3` to it using a register and the ALU, and written the updated value back - and the PC, having advanced on its own the whole time, is now sitting at 103, ready to fetch whatever instruction comes next. This same loop, with no new mechanism added, is also how a CPU runs a million-line program; it just repeats the loop a million-plus times, occasionally redirected by a branch instruction overwriting the PC mid-cycle instead of letting it advance sequentially.
