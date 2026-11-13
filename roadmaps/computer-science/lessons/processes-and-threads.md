---
title: "Processes and Threads"
description: What separates a process from a thread, why threads are cheaper to create and switch between, and the states a process moves through as it runs.
roadmap: computer-science
node: processes-and-threads
group: operating-systems
video:
  provider: youtube
  id: "ITc09gOrqZk"
  start: 0
  end: null
creator:
  name: Gate Smashers
  channel_url: https://www.youtube.com/@GateSmashers
duration_seconds: 676
level: beginner
prerequisites:
  - instruction-sets-and-assembly-basics
takeaways:
  - "A process is a running program plus everything the OS set aside for it: its own address space, open file handles, and a process control block the kernel uses to manage it."
  - "A thread is a unit of execution inside a process — it has its own program counter, registers, and stack, but shares the process's memory and file handles with every other thread in that process."
  - "Threads are cheaper than processes to create and switch between because creating one skips setting up a new address space, and switching between threads of the same process doesn't require the OS to swap memory-mapping hardware state."
  - "Because threads share memory, they can pass data to each other just by reading and writing shared variables — fast, but it opens the door to race conditions that separate processes never face."
  - "A process moves through a lifecycle of states — new, ready, running, waiting, and terminated — and the scheduler is the thing deciding when a process moves from ready to running."
  - "Two processes are isolated by default: a bug or crash in one normally cannot corrupt another process's memory, which is the main safety tradeoff against the speed of threads."
check_questions:
  - q: "Why is creating a new thread so much cheaper than creating a new process, when both end up running code on the CPU?"
    a: "Creating a process means the OS has to build a brand-new address space (page tables, a fresh heap and stack region, copies of open file descriptors), while creating a thread reuses the existing process's address space and only needs a new stack, program counter, and register set — far less setup work."
  - q: "If two threads in the same process can corrupt each other's data by both touching a shared variable, why not just always use separate processes to avoid that risk entirely?"
    a: "Separate processes are safer but far more expensive to create, switch between, and communicate across (since they don't share memory and need explicit IPC), so most programs accept the shared-memory risk in exchange for threads' speed and use synchronization tools to manage it."
  - q: "A process is sitting in the 'ready' state. What does that actually mean about what it's doing right now?"
    a: "It means the process has everything it needs to run and is just waiting for the scheduler to give it a turn on the CPU — it is not blocked on anything external, it simply isn't the process currently occupying a CPU core."
deeper:
  - label: "Difference between Process and Thread (GeeksforGeeks)"
    url: "https://www.geeksforgeeks.org/operating-systems/difference-between-process-and-thread/"
related_formulas: []
---

## A process is a program plus a world to run in

A program sitting on disk is just a file: instructions and data, inert. The moment you launch it, the operating system turns it into a **process** — the program loaded into memory, plus everything the OS tracks on its behalf so it can actually run safely alongside every other process on the machine. That bookkeeping includes a private **address space** (the process believes it owns the entire range of memory addresses it can see, even though the OS is really mapping that onto shared physical RAM), a table of **open file descriptors**, and a kernel data structure called the **process control block (PCB)** that stores the process's ID, its current state, its register values when it isn't running, and pointers to its memory and resources. Two processes cannot casually read or write each other's memory — if process A has a bug that corrupts a variable, process B's memory is untouched, because the hardware's memory-management unit enforces the boundary between their address spaces.

## A thread is a path of execution through that world

A **thread** lives inside a process and is the actual unit the CPU executes: it has its own program counter (where it currently is in the code), its own register values, and its own stack (for local variables and function calls). What it does *not* have is its own address space — every thread in a process shares that process's heap, global variables, and open files. A single-threaded process has exactly one path of execution; a multi-threaded process (a web browser rendering a page while downloading an image and listening for clicks) has several paths of execution running concurrently, all able to see and modify the same data.

This sharing is the entire reason threads exist. Spawning a new process means the OS allocates fresh page tables and memory regions — real, measurable setup cost. Spawning a new thread reuses the parent process's address space and only needs a new stack and register set, which is why thread creation is often an order of magnitude faster. The same logic applies to **context switching**: switching the CPU from one process to another requires the OS to swap out the memory-mapping hardware state (so the new process sees its own address space), which can mean flushing cached address translations. Switching between two threads *of the same process* skips that step entirely, since they already share the same address space — only the register set and stack pointer need to change. That's the mechanical reason threads are called "lightweight" and processes "heavyweight."

The tradeoff is isolation versus speed. Because threads share memory, passing data between them is just writing to a shared variable — no serialization, no OS-mediated channel. But that same shared access means two threads can race to update the same variable and corrupt it (the subject of the synchronization lesson later in this group), a failure mode that simply cannot happen between two separate processes unless they deliberately set up shared memory.

## The states a process moves through

Every process the OS manages sits in one of a small number of states, tracked in its PCB:

- **New** — the process is being created; the OS is setting up its PCB and memory.
- **Ready** — the process has everything it needs to run and is waiting only for the scheduler to assign it a CPU core.
- **Running** — the process is actually executing on a CPU right now.
- **Waiting (or blocked)** — the process can't proceed because it's waiting on something external, like a disk read finishing or data arriving on a network socket.
- **Terminated** — the process has finished (or been killed), and the OS is reclaiming its resources.

A process cycles between ready and running constantly on any multitasking system: the scheduler gives it a turn on the CPU (running), eventually takes the CPU back or the process needs I/O (waiting or back to ready), and the process re-enters the ready queue until it gets another turn. Threads have an analogous set of states, scoped to the thread rather than the whole process. Understanding this state machine is the foundation for the next lesson: CPU scheduling algorithms are exactly the policies that decide, among all the processes sitting in "ready," which one gets to move into "running" next.
