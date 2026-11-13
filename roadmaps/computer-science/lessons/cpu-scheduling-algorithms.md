---
title: "CPU Scheduling Algorithms"
description: How an OS picks which ready process runs next — FCFS, SJF, round robin, and priority scheduling — and the throughput-vs-fairness tradeoffs between them.
roadmap: computer-science
node: cpu-scheduling-algorithms
group: operating-systems
video:
  provider: youtube
  id: "TxjIlNYRZ5M"
  start: 0
  end: null
creator:
  name: Gate Smashers
  channel_url: https://www.youtube.com/@GateSmashers
duration_seconds: 983
level: beginner
prerequisites:
  - processes-and-threads
takeaways:
  - "A CPU scheduler picks one process from the ready queue to run next whenever the CPU is free, and the algorithm it uses directly determines how fairly and how quickly work gets done."
  - "First-come-first-served (FCFS) runs processes strictly in arrival order with no preemption, which is simple but lets one long process (the \"convoy effect\") make every process behind it wait."
  - "Shortest-job-first (SJF) runs whichever ready process needs the least CPU time next, which minimizes average waiting time but requires predicting burst length and can starve long processes."
  - "Round robin gives every ready process a fixed time slice (a quantum) in turn, which guarantees fairness and good response time but adds context-switch overhead and can hurt throughput if the quantum is too small."
  - "Priority scheduling runs the highest-priority ready process first, which lets important work jump the queue but can starve low-priority processes unless the OS ages their priority upward over time."
  - "No single algorithm wins on every metric at once — FCFS and SJF optimize for throughput and total wait time, round robin optimizes for response time and fairness, and priority scheduling optimizes for importance over fairness."
check_questions:
  - q: "Why does FCFS scheduling suffer from the 'convoy effect,' and why doesn't round robin have the same problem?"
    a: "Under FCFS a process keeps the CPU until it finishes, so if a long CPU-bound process is at the front of the queue, every short process behind it has to wait for the whole thing to complete; round robin avoids this because it forcibly interrupts any process after one time quantum, so a long process can never block short ones indefinitely."
  - q: "SJF minimizes average waiting time better than any other non-preemptive algorithm, so why isn't it used as the default scheduler in general-purpose operating systems?"
    a: "SJF requires knowing (or accurately predicting) each process's next CPU burst length in advance, which is generally unknowable for interactive, general-purpose workloads, and it can starve long processes indefinitely if a steady stream of short jobs keeps arriving."
  - q: "What happens to round robin's performance if the time quantum is set far too small, and what happens if it's set far too large?"
    a: "Too small a quantum means the OS spends a large fraction of its time on context switches rather than useful work, hurting throughput; too large a quantum means each process effectively runs to completion before the next gets a turn, making round robin behave just like FCFS and losing its fairness and response-time advantages."
deeper:
  - label: "Scheduling Algorithms – Solved Problems (Neso Academy)"
    url: "https://www.youtube.com/watch?v=g6QjVDyGgSE"
related_formulas: []
---

## The scheduler's job

Once a process is in the "ready" state (as covered in the previous lesson), it's just sitting in a queue, fully able to run but not currently assigned to a CPU core. Every time a core becomes free — a running process finishes, blocks on I/O, or gets preempted — the OS has to pick which ready process gets it next. That decision is made by the **CPU scheduler**, and the rule it follows is the **scheduling algorithm**. The choice of algorithm has real, measurable consequences: it changes how long processes wait, how quickly the system feels responsive to a user, and how much total work gets done per second.

To compare algorithms concretely, OS courses use a few standard metrics: **waiting time** (time spent in the ready queue, not running), **turnaround time** (total time from arrival to completion), **response time** (time from arrival to the *first* time the process gets the CPU), and **throughput** (processes completed per unit time). Different algorithms optimize different combinations of these, and improving one often costs another.

## First-come-first-served (FCFS)

FCFS is the simplest policy: processes run in the exact order they arrive in the ready queue, and once a process has the CPU, it keeps it until it finishes (non-preemptive). It's trivial to implement — a plain FIFO queue — but it has a well-known failure mode called the **convoy effect**: if a CPU-bound process that needs 50 seconds arrives first, every short process behind it (each needing only a few milliseconds) is stuck waiting the full 50 seconds, even though serving them first would have finished far more work in that time. Average waiting time under FCFS can be very sensitive to arrival order.

## Shortest-job-first (SJF)

SJF fixes exactly that problem by always picking, from the ready queue, whichever process has the shortest next CPU burst. Mathematically, SJF is provably optimal for minimizing average waiting time among non-preemptive algorithms — among a fixed set of jobs, running the shortest ones first front-loads completions and keeps the average wait lower than any other order. The catch is practical: the scheduler has to know how long a process will run *before* running it, which is generally impossible for general-purpose, interactive workloads (the OS doesn't know the future). Real systems approximate it by predicting a process's next burst from its recent history. SJF also has a fairness problem mirroring FCFS's: a long process can be starved indefinitely if short processes keep arriving ahead of it in line.

## Round robin (RR)

Round robin takes a completely different approach: instead of trying to guess what's optimal, it forces fairness. Every ready process gets a fixed **time quantum** (say, 20 milliseconds); when its quantum expires, it's preempted and sent to the back of the ready queue, and the next process gets a turn. No process can be blocked behind another for longer than roughly (quantum × number of ready processes). This gives round robin excellent response time — every process is guaranteed to get the CPU again soon — which is why it underlies most interactive, time-sharing systems. The tradeoff lives entirely in the quantum size. Too small, and the OS burns a large share of its time on context switches instead of useful work, hurting throughput. Too large, and round robin degenerates into FCFS, since each process effectively runs to completion before anyone else gets a turn. Picking a good quantum is a genuine engineering tradeoff, not a solved problem — it depends on the typical burst lengths in the workload.

## Priority scheduling

Priority scheduling assigns each process a priority number and always runs the highest-priority ready process. This is the right tool when some work genuinely matters more than fairness — a system alert handler should run before a background log compressor regardless of arrival order. It comes in preemptive and non-preemptive flavors, and it has the same starvation risk as SJF: a low-priority process can wait forever if higher-priority work keeps arriving. The standard fix is **aging** — gradually increasing a waiting process's priority the longer it sits in the ready queue, guaranteeing it eventually becomes the highest priority and runs.

## No free lunch

Put side by side, these four algorithms trace out a clear tradeoff triangle: FCFS and SJF chase throughput and low average wait time at the cost of fairness and responsiveness; round robin chases fairness and response time at some cost to throughput; priority scheduling chases "importance" at the cost of fairness unless aging is added. Real operating systems (Linux's CFS, Windows' scheduler) are hybrids that blend these ideas — using priority-like weighting on top of a round-robin-like time-sharing core — precisely because no single pure algorithm wins on every axis at once.
