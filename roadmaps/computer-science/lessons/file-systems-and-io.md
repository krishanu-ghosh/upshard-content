---
title: "File Systems and I/O"
description: How a file system organizes data on disk as inodes and directory trees, and why the OS buffers reads and writes instead of hitting disk every time.
roadmap: computer-science
node: file-systems-and-io
group: operating-systems
video:
  provider: youtube
  id: "KN8YgJnShPM"
  start: 0
  end: null
creator:
  name: CrashCourse
  channel_url: https://www.youtube.com/@crashcourse
duration_seconds: 723
level: beginner
prerequisites: [memory-management-and-virtual-memory]
takeaways:
  - "A file system's core job is tracking, for every file, which physical disk blocks hold its data — since a file's blocks are rarely contiguous, the system needs a structured way to find them all in order."
  - "An inode (or an entry in a file allocation table, depending on the file system) is the metadata record that points to a file's data blocks, directly, through indirect pointers, or both, so the file system can locate every block without storing them as one unbroken run."
  - "A directory is not a container of files in any physical sense — it's just another file holding a list of (name, inode-number) pairs, which is why directories naturally nest into a tree: a subdirectory is a file whose entries point to more inodes, some of which are themselves directories."
  - "Unbuffered I/O means every read or write call goes straight to the disk device, which is simple but makes small, frequent operations extremely slow because disk access latency dwarfs memory access latency."
  - "Buffered I/O inserts an in-memory cache between the application and the disk, so the OS can batch small writes into fewer, larger disk operations and serve repeated reads from memory instead of the disk, trading a small risk of data loss on a crash for a large gain in throughput."
check_questions:
  - q: "Why can't a file system just store a file's data in one contiguous run of disk blocks and record only the starting block and length?"
    a: "As files are created, grown, and deleted over time, free space on disk becomes fragmented, so a growing file often can't find one contiguous region large enough to hold it. Inodes (or file allocation table chains) solve this by letting a file's blocks be scattered anywhere on disk, with the metadata structure recording the scattered locations instead of relying on contiguity."
  - q: "If a directory is just a file, why does deleting a directory entry not necessarily delete the file's actual data?"
    a: "The directory entry is only a (name, inode-number) mapping; the inode and the data blocks it points to exist independently of any particular directory entry. A file's data is only actually freed once its last remaining link (directory entry) pointing to its inode is removed — which is exactly the mechanism behind hard links letting one file appear under multiple names or directories."
  - q: "Why does buffered I/O improve performance even for an application that only ever reads each byte of a file once?"
    a: "Disks are read in whole blocks regardless of how many bytes the application actually asked for, so even a single-byte read pulls in an entire block; buffering keeps that block in memory so that the application's next several small reads (which usually land in the same block) are served from memory instead of triggering a fresh disk access each time, batching many small logical reads into far fewer physical ones."
deeper:
  - label: "L-7.1: File System in Operating System | Windows, Linux, Unix, Android etc."
    url: "https://www.youtube.com/watch?v=0LtuQhNFFe0"
related_formulas: []
---

## The problem a file system solves

A disk, at the hardware level, is just a huge flat array of fixed-size blocks that can be read or written by number — it has no concept of "files" or "names" at all. A **file system** is the layer of software that imposes structure on top of that flat array: it decides which blocks belong to which file, gives files human-readable names, and organizes those names into directories. Everything that follows is really answering one question: given a file name, how does the OS find the actual bytes on disk?

## Tracking a file's blocks: inodes and file allocation tables

As a file grows and shrinks over its lifetime, and as other files are created and deleted around it, its data rarely ends up sitting in one tidy, contiguous run of disk blocks — free space gets fragmented, so a file's blocks are often scattered across the disk. The file system needs a structure that can point to all of a file's scattered blocks without wasting huge amounts of space itself.

Unix-style file systems solve this with an **inode** (index node): one inode per file, holding its metadata (size, permissions, timestamps) plus a set of pointers to its data blocks. A common design uses a hybrid scheme: a handful of direct pointers straight to the first few data blocks for small files, then an **indirect pointer** to a block that itself is full of pointers to further data blocks. Even **double** and **triple indirect pointers** — blocks of pointers to blocks of pointers — exist, letting a single, fixed-size inode address enormous files without the inode itself ever needing to grow. A small file touches only the direct pointers; a huge file leans on the indirect layers, paying one extra disk read per layer of indirection to find its data.

Older systems like FAT (File Allocation Table) take a different but related approach: rather than one inode per file holding pointers, there's a single table shared by the whole volume, where each entry says "the next block of this file is block N" (or marks end-of-file) — effectively a linked list of blocks threaded through a lookup table instead of through an inode's own pointer slots.

**Worked example:** say an inode has 10 direct pointers and blocks are 4 KB. A file under 40 KB is addressed entirely by direct pointers — one disk read gets its data after the inode is loaded. A 4 MB file needs roughly a thousand blocks, far more than 10 direct pointers can name, so most of it is reached through the single indirect pointer (a block full of pointers can hold hundreds of entries), and the file system pays one extra read to fetch that pointer block before it can fetch the data itself.

## Directories: files that point to other files

A directory is not a special on-disk structure — it's simply a file whose contents are a list of **(name, inode-number) pairs**. Looking up `/home/alice/notes.txt` means: read the root directory's data to find the inode number for `home`, load that inode, read *its* data (since it's a directory too) to find the inode number for `alice`, load that inode, and read its data to find the inode number for `notes.txt`. This is exactly why directories naturally form a **tree**: a directory entry can point to the inode of another directory just as easily as to the inode of an ordinary file, and nothing stops that nesting from continuing arbitrarily deep. It also explains hard links. Because a directory entry is just a pointer to an inode, the same inode can legitimately have multiple directory entries — even in different directories — pointing at it, and the underlying data is only freed once the last such entry is removed.

## Buffered vs. unbuffered I/O: why the OS doesn't hit disk every time

Even with an efficient block-pointer structure, physically accessing a disk is slow — orders of magnitude slower than accessing RAM, especially for spinning disks where a read means physically moving a head. If every `read()` or `write()` call an application makes translated directly into a disk operation (**unbuffered I/O**), a program that writes one line at a time to a log file would trigger one slow disk operation per line, and a program reading a file one byte at a time would trigger one disk operation per byte.

**Buffered I/O** fixes this by inserting an in-memory cache (maintained by the OS, and often duplicated by a language's standard library on top of that) between the application and the actual disk device. Writes accumulate in the buffer and only get flushed to disk once the buffer fills, a timer elapses, or the application explicitly asks for a flush — turning many small writes into one large, efficient disk operation. Reads work the same way in reverse: a read pulls in a whole block even if the caller only asked for a few bytes, so subsequent nearby reads are served straight from memory instead of going back to disk.

The trade-off is durability: data sitting in a buffer that hasn't been flushed yet is vulnerable to a crash or power loss, which is why operations that must survive a crash (database commit logs, for instance) explicitly force a flush (`fsync`) rather than trusting the buffer's normal schedule. For almost everything else, the throughput gained from batching is well worth that small window of risk.
