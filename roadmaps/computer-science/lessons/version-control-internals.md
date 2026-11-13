---
title: "Version Control Internals: How Git Actually Stores Data"
description: Git's real mental model — commits as content-addressed snapshots forming a DAG, and branches as nothing more than movable pointers.
roadmap: computer-science
node: version-control-internals
group: concurrency-and-practices
video:
  provider: youtube
  id: "lG90LZotrpo"
  start: 0
  end: null
creator:
  name: CS50
  channel_url: https://www.youtube.com/@cs50
duration_seconds: 3459
level: intermediate
prerequisites: []
takeaways:
  - "A Git commit stores a full snapshot of the entire project tree at that point in time, not a diff against the previous commit — diffs are computed on demand for display, never stored."
  - "Every object Git stores — blob (file contents), tree (a directory listing), and commit — is identified by the SHA-1/SHA-256 hash of its own content, so identical content is always stored exactly once."
  - "Commits link to their parent commit(s) by hash, which makes the full history a directed acyclic graph (DAG): it can branch and merge but, because each commit references the past, it can never cycle back on itself."
  - "A branch is just a small text file holding one commit's hash — moving a branch forward (a new commit, a merge) is just rewriting that one pointer, which is why creating or switching branches is instant."
  - "HEAD is a pointer to whichever branch (or commit) you currently have checked out, and `git commit` works by creating a new commit object, then moving the current branch pointer to point at it."
  - "Because every object is content-addressed, changing anything in a commit changes its hash, which changes every descendant commit's hash too — this is why Git history is effectively tamper-evident."
check_questions:
  - q: "If Git stores a full snapshot in every single commit, why don't repositories with thousands of commits become enormous?"
    a: "Snapshots are made of blobs and trees that are content-addressed by hash, so a commit that only changes one file reuses the exact same blob and tree objects for every unchanged file and directory from the previous commit — only the handful of objects that actually changed, plus the new commit object itself, get stored."
  - q: "Why is switching branches in Git nearly instant, even in a huge repository?"
    a: "A branch is just a pointer (a file containing a commit hash) — switching branches means updating HEAD to point at a different branch and then updating the working directory's files to match that commit's tree, which doesn't require rewriting any history, only reading the objects that make up that one snapshot."
  - q: "Two developers each commit on their own branch starting from the same point, then merge. Why does the resulting history look like it 'branches and rejoins' rather than being a straight line?"
    a: "Each commit records its parent(s) by hash, and since both developers' commits share the same starting parent, the graph forks there; a merge commit then records two parents — one from each branch — which rejoins the fork into a single node without erasing either path, producing the DAG shape you see in a visual history."
deeper:
  - label: "Git Internals — Git Objects (Pro Git book)"
    url: "https://git-scm.com/book/en/v2/Git-Internals-Git-Objects"
related_formulas: []
---

## The model everyone's taught wrong

Most people learn Git as a sequence of memorized commands — `add`, `commit`, `push`, `merge`, `rebase` — without ever being shown what's actually happening underneath, which is why Git commands feel arbitrary and merge conflicts feel like magic going wrong. The commands make sense immediately once you know one thing: Git is a content-addressed object store with a graph of snapshots sitting on top of it. Everything else — branches, tags, merges, `git log` — is just a thin layer of pointers into that store.

## Commits are snapshots, not diffs

The single most common misconception is that Git stores each commit as a diff against the one before it, the way some older version control systems did. It doesn't. **Every commit stores a complete snapshot of the entire project's file tree at that moment.** When you run `git commit`, Git doesn't compute what changed relative to the parent commit and save just that; it writes out a full tree of the current state and an object representing the commit itself, which points at that tree and at the commit's parent (or parents).

This seems wasteful until you see the second half of the model: Git doesn't actually duplicate unchanged files between snapshots, because of how it identifies objects.

## Content addressing: the hash is the identity

Git stores three main kinds of objects:

- A **blob** holds the raw contents of a single file (just the bytes — no filename, no permissions).
- A **tree** is a directory listing: a set of entries, each naming a file or subdirectory and pointing at the hash of the blob or tree that represents it.
- A **commit** points at one root tree (the snapshot) plus zero or more parent commits, along with metadata — author, timestamp, message.

Every one of these objects is named by the cryptographic hash (SHA-1, with newer repositories moving to SHA-256) of its own content. This is **content addressing**: an object's identity isn't assigned by Git, it's derived from what's inside it. Two files with byte-for-byte identical content hash to the same value and are therefore stored as the exact same blob, even in totally unrelated parts of the repository or history. This is also why a commit that only edits one file out of ten thousand is cheap: the new commit's tree reuses the hashes of the nine thousand nine hundred ninety-nine unchanged blobs and the subtrees that didn't need to change, and only writes new objects for the file that changed, the trees on the path down to it, and the new commit itself.

It's also why Git history is tamper-evident: a commit's hash is computed over its content, which includes its parent's hash. Change anything in an old commit — even one character of one file — and that commit's hash changes, which changes the tree hash above it, which changes every commit that descends from it. You can't quietly edit history; you can only create new, differently-hashed objects and move pointers to them, which is exactly what operations like `rebase` and `amend` actually do under the hood (they don't edit old commits — they build new ones and discard the old references).

## The DAG: branching and merging without cycles

Because each commit stores the hash of its parent (or, for a merge, two parents), the full set of commits forms a **directed acyclic graph** — directed because each edge points from a commit to its parent, acyclic because a commit can never be its own ancestor (you can't create a commit before its parent exists, so time only flows one way through the graph). A normal commit has one parent, extending a single line. A merge commit has two parents, joining two previously separate lines of history into one node without deleting either line — which is exactly why a repository's history can fork into feature branches and rejoin later, visually branching and reconverging, while remaining a strict DAG with no loops.

## A branch is just a pointer — nothing more

This is the part that makes everyday Git commands click: a branch, like `main` or `feature-x`, is not a copy of anything, not a separate timeline with its own storage. It is a tiny file (under `.git/refs/heads/`) containing exactly one thing: the hash of the commit it currently points to. `HEAD` is, in turn, usually a pointer to whichever branch you have checked out.

Once you see branches this way, the commands stop being arbitrary:

- `git branch new-feature` creates a new 40-character text file pointing at your current commit — instant, because it copies one hash, not any history.
- `git checkout` / `git switch` updates `HEAD` to point at a different branch (or commit), then updates your working files to match whatever snapshot that commit's tree describes.
- `git commit` creates a new commit object whose parent is the commit your current branch points to, then moves your current branch's pointer forward to the new commit's hash.
- Deleting a branch just deletes that one pointer file — the commits it pointed to aren't erased immediately; they simply become unreachable by name and are eventually garbage-collected if nothing else references them.

Every Git operation you'll ever run is some combination of: write a new content-addressed object, and/or move a pointer to point at a different commit. Once that's the mental model, the commands you memorized stop being incantations and start being exactly what they say they are.
