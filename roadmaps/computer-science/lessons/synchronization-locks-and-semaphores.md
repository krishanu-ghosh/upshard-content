---
title: "Synchronization: Locks and Semaphores"
description: Why concurrent access to shared data needs coordination, how mutexes and semaphores provide it, and how both solve the classic producer-consumer problem.
roadmap: computer-science
node: synchronization-locks-and-semaphores
group: operating-systems
video:
  provider: youtube
  id: "hh9g5kKl_aE"
  start: 0
  end: null
creator:
  name: Gate Smashers
  channel_url: https://www.youtube.com/@GateSmashers
duration_seconds: 1050
level: beginner
prerequisites:
  - cpu-scheduling-algorithms
takeaways:
  - "When two threads can read and write the same memory, the scheduler can interleave their instructions in ways that corrupt shared data — this is a race condition, and it exists precisely because threads share an address space."
  - "A mutex (mutual exclusion lock) lets only one thread at a time execute a critical section — a thread must acquire the lock before entering and release it after, and any other thread trying to acquire it blocks until it's free."
  - "A semaphore is a counter with two atomic operations, wait (decrement, blocking if the result would go negative) and signal (increment); a semaphore initialized to 1 behaves like a mutex, but a semaphore can also be initialized to N to let up to N threads through at once."
  - "Counting semaphores are the natural tool for signaling availability of a resource pool, not just for mutual exclusion — the producer-consumer problem uses exactly this to track empty and full slots in a shared buffer."
  - "The producer-consumer problem needs three synchronization primitives working together: a mutex to protect the buffer itself, and two counting semaphores — one counting empty slots, one counting full slots — to make producers and consumers block instead of busy-waiting when the buffer is full or empty."
  - "Even with correct locking, a thread that acquires a lock and never releases it (due to a bug or a crash) can freeze every other thread waiting on it — the reason real systems favor lock patterns (like RAII or try/finally) that guarantee release."
check_questions:
  - q: "If a simple read-modify-write operation like incrementing a shared counter can be interrupted mid-way by the scheduler, why does that cause the final count to be wrong?"
    a: "Incrementing isn't one CPU instruction — it reads the current value, adds one, then writes it back — so if the scheduler switches to another thread between the read and the write, that thread can read the same stale value, and when both threads eventually write back, one thread's update silently overwrites the other's, losing an increment."
  - q: "A mutex is really just a semaphore initialized to 1. What capability does a general counting semaphore give you that a mutex (binary semaphore) can't?"
    a: "A mutex only ever allows one thread through at a time, but a counting semaphore initialized to N allows up to N threads to proceed concurrently, which is exactly what's needed to represent a pool of N interchangeable resources (like N empty buffer slots) rather than a single exclusive one."
  - q: "In the producer-consumer solution, why does the producer wait on the 'empty' semaphore before acquiring the mutex, rather than acquiring the mutex first?"
    a: "If the producer grabbed the mutex first and then found the buffer full, it would have to block while still holding the mutex, which would prevent the consumer from ever acquiring the mutex to remove an item and free up space — a deadlock; waiting on 'empty' first ensures the producer only takes the mutex when it already knows there's room to act."
deeper:
  - label: "L-3.8: Semaphores | Wait, Signal Operation | Counting Semaphore (Gate Smashers)"
    url: "https://www.youtube.com/watch?v=eoGkJWgxurQ"
related_formulas: []
---

## Why shared memory needs rules

The previous lesson's threads share a process's memory precisely because that's what makes them cheap and useful — but sharing cuts both ways. If two threads both read a shared variable, modify it, and write it back without coordination, the scheduler is free to interleave their instructions however it likes (that's the whole point of preemptive scheduling). Suppose two threads both execute `balance = balance + 10` on a shared bank balance starting at 100. Each thread actually performs three steps: read `balance` into a register, add 10, write the register back to `balance`. If thread A reads 100, then the scheduler switches to thread B before A writes back, B also reads 100, adds 10, and writes 110. When A resumes, it still has 100 in its register, adds 10, and writes 110 — one deposit has vanished, even though both increments "happened." This is a **race condition**: the final result depends on scheduling timing rather than program logic, and the segment of code where it occurs (`balance = balance + 10` here) is called a **critical section**.

## Mutexes: one at a time

The most direct fix is to make the critical section exclusive. A **mutex** (mutual exclusion lock) is an object with two operations: `lock()` and `unlock()`. A thread must call `lock()` before entering the critical section; if another thread already holds the lock, the caller blocks until it's released. Once inside, the thread does its read-modify-write safely, then calls `unlock()` to let the next waiting thread in. With a mutex around `balance = balance + 10`, the interleaving above becomes impossible — B simply cannot start its read until A has fully finished its write and released the lock. The discipline this requires is simple to state and easy to get wrong in practice: every path out of the critical section, including early returns and exceptions, must release the lock, or every other thread waiting on it is stuck forever.

## Semaphores: a counter with blocking built in

A **semaphore** generalizes the same idea into a counter with two atomic operations, traditionally called `wait()` (or P) and `signal()` (or V). `wait()` decrements the counter; if the result would go negative, the calling thread blocks instead. `signal()` increments the counter and wakes a blocked thread if one is waiting. A semaphore initialized to 1 behaves exactly like a mutex — only one `wait()` can succeed before a `signal()` — which is why a semaphore initialized to 1 is sometimes called a **binary semaphore**. The real power shows up when you initialize it to N: a **counting semaphore** lets up to N threads proceed before any of them blocks, which is the natural way to represent a pool of N interchangeable resources rather than a single exclusive one — N available database connections, N free buffer slots, and so on.

## Worked example: producer-consumer

The producer-consumer problem is the standard illustration of why you often need *both* tools together. A producer thread generates items and places them into a fixed-size shared buffer; a consumer thread removes items from that buffer and processes them. Two things can go wrong: the producer shouldn't write into a full buffer, and the consumer shouldn't read from an empty one — and neither should corrupt the buffer's internal bookkeeping (its read/write indices) by acting at the same time as the other.

The standard solution uses three primitives:

- `mutex` — a binary semaphore (or plain mutex) protecting the buffer's indices themselves.
- `empty` — a counting semaphore initialized to the buffer's capacity N, tracking how many empty slots remain.
- `full` — a counting semaphore initialized to 0, tracking how many filled slots are ready to consume.

The producer calls `wait(empty)` first — blocking here if the buffer is completely full — *then* `wait(mutex)`, writes the item, calls `signal(mutex)`, and finally `signal(full)` to tell a waiting consumer an item is ready. The consumer mirrors it: `wait(full)` (blocking if nothing is available), `wait(mutex)`, remove the item, `signal(mutex)`, `signal(empty)`.

The order matters: `wait(empty)` or `wait(full)` always comes *before* `wait(mutex)`. If a producer acquired the mutex first and then discovered the buffer full, it would block while still holding the mutex — and the consumer, needing that same mutex to remove an item and free a slot, could never get in. That's a deadlock, and it's exactly the failure mode the next lesson covers in depth. Getting the acquisition order right here is what makes this "classic" problem a genuinely instructive one: it's not just about mutual exclusion, it's about using counting semaphores to make threads wait for the *right condition*, not just for each other.
