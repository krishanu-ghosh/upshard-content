---
title: "Interprocess Communication"
description: How separate processes that don't share memory exchange data — pipes, message queues, shared memory, and sockets — and when to reach for each.
roadmap: computer-science
node: interprocess-communication
group: operating-systems
video:
  provider: youtube
  id: "dJuYKfR8vec"
  start: 0
  end: null
creator:
  name: Neso Academy
  channel_url: https://www.youtube.com/@nesoacademy
duration_seconds: 769
level: beginner
prerequisites: [processes-and-threads]
takeaways:
  - "Each process has its own private virtual address space, so one process cannot simply read or write another process's memory — they need the OS's cooperation to exchange any data at all."
  - "A pipe is a one-directional byte stream connecting two related processes (like a parent and its child), commonly used to chain commands where one process's output feeds another's input."
  - "A message queue lets unrelated processes exchange discrete, structured messages through a kernel-managed queue, so a sender can post a message and move on without the receiver needing to be listening at that exact instant."
  - "Shared memory maps the same physical frames into two or more processes' address spaces, making it the fastest IPC mechanism because, once set up, reading and writing involves no kernel intervention at all — but it pushes the burden of synchronization onto the application."
  - "Sockets extend IPC across machine boundaries, not just processes on the same box, by wrapping communication in a network-style addressing and protocol layer even when both endpoints happen to be local."
  - "There's a real trade-off between IPC mechanisms: shared memory is fastest but requires manual synchronization; message passing (pipes, queues, sockets) is slower because the kernel copies data on every exchange, but it comes with built-in safety since processes never touch each other's memory directly."
check_questions:
  - q: "Why can't two processes just exchange data by one of them reading a variable at a known memory address in the other?"
    a: "Each process has its own virtual address space and its own page table, so the same virtual address in two different processes maps to two different physical frames (or one is simply unmapped in the other). Without the OS explicitly setting up a shared mapping, there is no physical memory both processes can see, so a direct read is impossible."
  - q: "Why is shared memory the fastest IPC mechanism, and what does that speed cost the programmer?"
    a: "After the initial setup, both processes have the same physical frames mapped into their own address spaces, so reading or writing is a plain memory access with no system call and no data copying through the kernel. The cost is that the OS no longer mediates access, so the application itself must handle synchronization (e.g., with semaphores) to prevent both processes from reading and writing the same memory at the same time and corrupting it."
  - q: "If pipes are simpler and built into the shell, why would a system ever use a message queue instead?"
    a: "A pipe is typically restricted to related processes (like a parent and child created via fork) and delivers an undifferentiated byte stream with no message boundaries. A message queue lets unrelated processes communicate, preserves discrete message boundaries (and can prioritize messages), and lets a sender post a message that sits in the queue until the receiver is ready, rather than requiring both ends to be actively connected."
deeper:
  - label: "Inter Process Communication (IPC) in Operating System"
    url: "https://www.youtube.com/watch?v=-GUSNnRt2U8"
related_formulas: []
---

## Why IPC has to exist at all

The memory management lesson established that every process gets its own isolated virtual address space — that isolation is what keeps a crash or bug in one program from corrupting another. But isolation has a cost: processes that genuinely need to cooperate (a shell piping output between commands, a web server handing work to worker processes, a database serving many client connections) have no built-in way to pass data to each other. **Interprocess communication (IPC)** is the collection of OS-provided mechanisms that punch controlled holes in that isolation so cooperating processes can exchange data safely.

Every IPC mechanism makes a trade-off along the same axis: how much the kernel mediates the exchange. More mediation means more safety and more built-in structure, but more overhead per exchange. Less mediation means raw speed, but the application has to handle correctness itself.

## Pipes: a byte stream between related processes

A **pipe** is the simplest mechanism: the kernel allocates a fixed-size buffer and gives one process a write end and another a read end. Data written at one end comes out the other end in the same order, as an undifferentiated stream of bytes — there's no concept of "messages," just a stream, much like a file. Pipes are typically anonymous and scoped to related processes, most commonly a parent and a child it creates with `fork()`, because the file descriptors for the pipe's two ends have to be inherited across that fork. This is exactly what happens when you run `ls | grep foo` in a shell: the shell creates a pipe, then forks two child processes, wiring `ls`'s standard output to the pipe's write end and `grep`'s standard input to its read end. Neither program needs to know a pipe is involved; each just reads or writes its standard file descriptors as usual.

## Message queues: structured, decoupled messaging

A **message queue** is a kernel-managed queue of discrete, typed messages that unrelated processes can send to and receive from by referencing a shared queue identifier, rather than needing an inherited file descriptor. Because each message keeps its boundaries (unlike a pipe's raw byte stream) and the queue itself buffers messages, the sender and receiver don't need to be active at the same moment — a process can post a message and exit, and the receiver picks it up whenever it next checks the queue. Some implementations also support message priorities, letting urgent messages jump ahead of routine ones. The cost is still a kernel round-trip (and a data copy) per message, since the kernel owns the queue's storage.

## Shared memory: cutting the kernel out of the hot path

**Shared memory** takes the opposite approach: instead of funneling every exchange through the kernel, the OS maps the *same physical frames* into the virtual address space of two or more processes, using exactly the page-table mechanism from the memory management lesson. Once that mapping exists, process A writing to its mapped region is physically writing to the same RAM that process B reads from its own mapped region — no system call, no copying, no kernel involvement on the read or write itself. This makes shared memory the fastest IPC mechanism by a wide margin, which is why high-throughput systems (database engines, media pipelines) reach for it. The price is that the OS no longer polices access: if both processes write at once without coordination, the data can end up corrupted or torn, so the application must supply its own synchronization, typically semaphores, to take turns safely.

## Sockets: IPC that also works across the network

A **socket** is an endpoint for communication identified by an address (and, for networked sockets, a port), and it's deliberately built so the same API works whether the other endpoint is another process on the same machine (a Unix domain socket) or a process on a different machine entirely (a TCP/IP socket). This generality costs a bit more overhead than a pipe for same-machine communication, but it's the only one of these four mechanisms that scales to processes that aren't even on the same computer, which is why it's the backbone of client-server and networked systems.

## Picking one

For a simple producer feeding a consumer in a shell pipeline: a pipe. For decoupled, structured messages between unrelated processes that don't need to talk constantly: a message queue. For maximum throughput between processes on the same machine that can manage their own locking: shared memory. For anything that might someday cross a network boundary, or already needs addressable, connection-oriented communication: a socket.
