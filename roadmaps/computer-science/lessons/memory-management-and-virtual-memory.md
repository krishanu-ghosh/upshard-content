---
title: "Memory Management and Virtual Memory"
description: Why every process gets its own private address space, how paging maps it to real RAM, and what happens when memory runs out.
roadmap: computer-science
node: memory-management-and-virtual-memory
group: operating-systems
video:
  provider: youtube
  id: "B6tJxvYBNrU"
  start: 0
  end: null
creator:
  name: BitLemon
  channel_url: https://www.youtube.com/@BitLemonSoftware
duration_seconds: 700
level: beginner
prerequisites: [memory-hierarchy-and-caching, processes-and-threads]
takeaways:
  - "Every process sees its own private virtual address space starting at address zero, so it never has to know or care where it actually sits in physical RAM."
  - "Virtual memory is divided into fixed-size pages, and physical RAM is divided into same-sized frames; a per-process page table maps each page to whichever frame currently holds it."
  - "Because the mapping is a layer of indirection, the OS can place a process's pages anywhere in RAM, move them around, and keep two processes from ever touching each other's memory."
  - "A page fault happens when the CPU asks for a virtual address whose page table entry has no frame assigned; the OS then loads that page from disk before execution can continue."
  - "When physical memory is oversubscribed, the OS swaps rarely-used pages out to disk to free frames for pages that are needed right now, trading disk I/O time for the ability to run more than RAM alone could hold."
  - "Demand paging means pages are only loaded when actually touched, not when a process starts, which is why large programs can launch quickly even though most of their code never runs in a given session."
check_questions:
  - q: "Why does giving every process its own virtual address space, rather than letting processes address physical RAM directly, make memory protection possible?"
    a: "A process can only name addresses within its own virtual space, and the page table is the only thing that translates those into real RAM locations; since the OS controls every process's page table and never maps two processes' pages onto the same frame (unless deliberately shared), one process has no way to even express, let alone touch, an address inside another process's memory."
  - q: "Why does the OS divide memory into fixed-size pages instead of just handing out physical memory in variable-sized chunks per allocation?"
    a: "Fixed-size pages make every unit interchangeable: a page table entry is a simple, uniform lookup, any free frame can hold any page, and the OS never has to deal with hunting for a contiguous block of exactly the right size. Variable-sized chunks lead to external fragmentation, where there's enough total free memory but no single gap large enough for a new allocation."
  - q: "If RAM is already full, why does triggering a page fault to load a missing page actually work rather than just failing?"
    a: "The OS first evicts a currently resident page (writing it back to disk if it was modified) to free up a frame, then loads the needed page into that freed frame. This is swapping — it's slower than if the page were already resident, but it lets the system run working sets larger than physical RAM at the cost of disk I/O."
deeper:
  - label: "L-5.19: Virtual Memory | Page Fault | Significance of Virtual Memory"
    url: "https://www.youtube.com/watch?v=o2_iCzS9-ZQ"
related_formulas: []
---

## The problem: processes can't share one flat address space safely

If every running process addressed physical RAM directly, chaos would follow almost immediately. Two programs compiled to assume they start at address `0x1000` would collide the moment both ran at once. A buggy or malicious process could read or overwrite another process's memory just by naming the right address. And the OS would have to relocate a program's entire memory footprint every time it needed to free up a contiguous block elsewhere.

Virtual memory solves this with a layer of indirection: every process gets its own **virtual address space**, typically starting at address zero and spanning the full range the CPU can address. A process only ever issues virtual addresses. Something else — the **memory management unit (MMU)**, working with OS-maintained tables — translates each one into a physical RAM address before the memory access actually happens. The process never sees physical addresses at all, so two processes can both believe they own address `0x1000` without any conflict, because each one's `0x1000` maps through a completely different table to a completely different physical location.

## Paging: carving memory into interchangeable units

The mapping between virtual and physical addresses is organized through **paging**. Virtual address space is divided into fixed-size chunks called **pages** (4 KB is a common size), and physical RAM is divided into same-sized chunks called **frames**. A **page table**, one per process, records which frame (if any) currently holds each page.

Fixed size is what makes this efficient: since every page and every frame is identical in size, any page can be placed in any free frame, and the hardware can split a virtual address mechanically into a page number and an offset within the page — look up the page number in the page table to get a frame number, then append the same offset to find the exact byte in physical RAM. There's no searching for a correctly-sized free region, which is the problem that plagued older variable-sized allocation schemes and caused external fragmentation.

**Worked example:** suppose pages are 4 KB (4096 bytes, so 12 bits of offset), and a process accesses virtual address `0x00003A10`. The low 12 bits (`0xA10`) are the offset within the page; the remaining high bits (`0x3`) identify page number 3. The CPU looks up entry 3 in that process's page table. If it says "frame 7," the physical address becomes frame 7's base address plus offset `0xA10`. If entry 3 instead has an invalid or not-present bit set, the hardware raises a **page fault** instead of completing the access.

## Page faults and swapping: what happens when the page isn't there

A page fault isn't necessarily an error — it's often the OS's cue to act. When a page fault occurs, the OS's fault handler checks why: the page might simply not be loaded yet (common in **demand paging**, where pages are only pulled in from disk the first time they're actually touched, rather than all at once when the program starts), or the access might genuinely be illegal (a true segmentation fault).

If the page is legitimate but not resident, the OS needs to get it into a frame. When free frames exist, it loads the page from disk and updates the page table entry, then re-runs the faulting instruction as if nothing happened. But when physical memory is fully committed across all running processes, the OS has to make room first: it picks a resident page to evict (using a replacement policy like least-recently-used), writes it back to disk if it's been modified, marks its frame free, and only then loads the needed page in. This whole mechanism — moving pages between RAM and disk to let total memory demand exceed physical RAM — is **swapping**.

Swapping is a release valve, not a free lunch: disk access is orders of magnitude slower than RAM, so a system that's swapping heavily (**thrashing**) can spend more time moving pages in and out than doing useful work. This is exactly why the prior lesson on the memory hierarchy matters here — virtual memory is effectively treating disk as one more, much slower, tier beneath RAM, and the OS's job is to keep the pages you're actively using in the fast tier as much as possible.
