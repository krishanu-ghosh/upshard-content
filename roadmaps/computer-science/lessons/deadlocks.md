---
title: "Deadlocks"
description: The four conditions that must all hold for a deadlock to occur, how a resource-allocation graph detects one, and how prevention and avoidance strategies work.
roadmap: computer-science
node: deadlocks
group: operating-systems
video:
  provider: youtube
  id: "rWFH6PLOIEI"
  start: 0
  end: null
creator:
  name: Gate Smashers
  channel_url: https://www.youtube.com/@GateSmashers
duration_seconds: 741
level: beginner
prerequisites:
  - synchronization-locks-and-semaphores
takeaways:
  - "A deadlock is a state where a set of processes are each waiting for a resource held by another process in the same set, so none of them can ever proceed."
  - "Deadlock requires all four of these conditions to hold simultaneously: mutual exclusion, hold-and-wait, no preemption, and circular wait — breaking any single one makes deadlock impossible."
  - "A resource-allocation graph models processes and resources as nodes with request and assignment edges; a cycle in that graph (with single-instance resources) is both necessary and sufficient proof that a deadlock exists."
  - "Deadlock prevention works by structurally ruling out one of the four necessary conditions, often at the cost of resource utilization — for example, requiring a process to request all its resources at once eliminates hold-and-wait but wastes resources it won't use yet."
  - "Deadlock avoidance, unlike prevention, allows all four conditions to be possible but uses information about future resource needs (as in the Banker's algorithm) to refuse any request that would risk an unsafe state."
  - "Detecting a deadlock after the fact and recovering (by preempting a resource or killing a process) is a third strategy — cheaper to run day-to-day than avoidance, but it means deadlocks are allowed to actually happen before anything is done about them."
check_questions:
  - q: "If you could magically guarantee that no process ever holds one resource while waiting for another, would deadlock still be possible? Why?"
    a: "No — that eliminates hold-and-wait, and since all four conditions (mutual exclusion, hold-and-wait, no preemption, circular wait) must hold at once for deadlock to occur, removing any single one makes the cycle of mutual waiting that defines deadlock impossible, regardless of what the other three conditions look like."
  - q: "Why is a cycle in the resource-allocation graph only a necessary-and-sufficient proof of deadlock when every resource type has a single instance?"
    a: "With single-instance resources, a cycle means every process in that cycle is waiting on a resource held by exactly one other process in the cycle, so none can ever be freed; but if a resource type has multiple instances, a cycle can exist while a different instance of that resource is still available to break it, so the system isn't necessarily stuck."
  - q: "The Banker's algorithm refuses a resource request even when the resources requested are currently available. What is it actually protecting against?"
    a: "It's checking whether granting the request could still leave the system in a 'safe state' — one where some ordering of processes could finish by having their maximum future needs met — so it refuses requests that would leave no such safe ordering, even though nothing is wrong at that exact instant; it's trading some resource utilization for a mathematical guarantee that deadlock can never actually arise."
deeper:
  - label: "Introduction of Deadlock in Operating System (GeeksforGeeks)"
    url: "https://www.geeksforgeeks.org/operating-systems/introduction-of-deadlock-in-operating-system/"
related_formulas: []
---

## When waiting never ends

The producer-consumer solution in the previous lesson showed how getting lock-acquisition order wrong can make a producer and a consumer wait on each other forever. That failure mode has a name: **deadlock** — a state where a set of processes (or threads) are each blocked waiting for a resource that another process in the same set is holding, and none of them will ever release anything, because releasing requires first acquiring the thing they're stuck waiting for. The classic illustration is two processes, each holding one resource the other needs: process A holds lock 1 and waits for lock 2; process B holds lock 2 and waits for lock 1. Neither will ever proceed, and no amount of waiting fixes it — this is qualitatively different from a process just being slow.

## The four necessary conditions

Deadlock research (going back to Coffman's 1971 analysis) identified four conditions that must *all* hold simultaneously for a deadlock to be possible. This matters practically: if you can engineer a system so that even one of these is structurally impossible, deadlock becomes impossible too, no matter how the scheduler interleaves things.

1. **Mutual exclusion** — at least one resource is held in a non-shareable way (only one process can use it at a time, like a lock or a printer).
2. **Hold-and-wait** — a process holding at least one resource is waiting to acquire additional resources currently held by other processes.
3. **No preemption** — a resource can only be released voluntarily by the process holding it; the OS cannot forcibly take it back.
4. **Circular wait** — there exists a cycle of processes, where each is waiting for a resource held by the next one in the cycle.

Notice these aren't independent observations about a specific bug — they're a checklist. A system that enforces "always request every resource you'll ever need up front, all at once" removes hold-and-wait. A system where resources *can* be forcibly reclaimed (like a CPU core, which the scheduler preempts routinely) removes "no preemption" for that resource type. If you can remove any one of the four from your system's design, you've made deadlock structurally impossible for the resources that policy covers.

## Proving it with a graph

The **resource-allocation graph** turns this into something checkable. Processes and resource instances are both nodes; an edge from a resource to a process means that resource is currently assigned to the process, and an edge from a process to a resource means the process is requesting (waiting for) it. For resource types with a single instance each, a cycle in this graph is both necessary and sufficient to prove a deadlock exists: every process in the cycle is waiting on a resource held by the very next process in the cycle, and since none of them can proceed to release what they hold, the wait is permanent. With multiple instances of the same resource type, a cycle is necessary but no longer sufficient. The cycle might still be breakable if a different, available instance of that resource type can satisfy one of the waiting processes, so systems with multi-instance resources need a more general detection algorithm that reasons about resource instance counts, not just the graph shape.

## Three ways to deal with it

**Prevention** attacks one of the four conditions directly, as described above. It's reliable but often wasteful — forcing a process to request all resources upfront means it's holding things it won't touch for a while, lowering overall utilization.

**Avoidance** is subtler: it allows all four conditions to remain possible in principle, but uses extra information to refuse any individual request that *could* lead to deadlock. The best-known example is the **Banker's algorithm**: each process declares its maximum possible future resource need in advance, and before granting any request, the OS checks whether the resulting state is still "safe" — meaning there exists at least one order in which every process could still finish, assuming each eventually asks for its stated maximum. If granting a request would leave no safe ordering, the OS makes the process wait, even though the resources requested are technically available right now.

**Detection and recovery** doesn't try to stop deadlock from happening at all — it lets the system run normally, periodically checks the resource-allocation graph for cycles, and when one is found, recovers by forcibly preempting a resource from one of the processes involved or terminating a process in the cycle outright. This is cheaper to run day-to-day than avoidance, since there's no per-request safety check, but it accepts that a real deadlock, with real processes frozen, happens before anything is done about it. Many general-purpose operating systems accept that tradeoff because true multi-resource deadlocks are comparatively rare in practice.
