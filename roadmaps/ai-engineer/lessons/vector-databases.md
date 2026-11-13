---
title: Vector databases
description: What a vector index actually does, how HNSW and IVF trade accuracy for speed, and when Postgres with pgvector is enough.
roadmap: ai-engineer
node: vector-databases
group: retrieval
video:
  provider: youtube
  id: "Fq73E8DQprc"
  start: 0
  end: 1219
creator:
  name: Hello Interview
  channel_url: https://www.youtube.com/@hello_interview
duration_seconds: 1473
level: intermediate
prerequisites: [embeddings]
takeaways:
  - Exact nearest-neighbour search compares the query against every vector, which is correct and too slow past a modest corpus size.
  - Approximate indexes such as HNSW and IVF trade a little recall for a large speed gain, and that trade is a tunable parameter rather than a fixed property.
  - Metadata filtering interacts badly with approximate search, because filtering after the search can leave you with far fewer results than you asked for.
check_questions:
  - q: What does the approximate in approximate nearest neighbour actually give up?
    a: Recall. The index may miss some true nearest neighbours in exchange for searching a fraction of the corpus, and how much it misses is tunable.
  - q: Why is filtering by metadata harder than adding a WHERE clause?
    a: An approximate index finds candidates by proximity, not by your filter. Filtering afterwards can discard most of them, so the engine has to push the constraint into the traversal to return a full result set.
deeper:
  - label: "Attila Toth — Why use PostgreSQL and pgvector instead of a specialised vector database"
    url: https://www.youtube.com/watch?v=EZ5KtRgRj34
  - label: "Mustafa Zaki — Vector search and ANN with FAISS, HNSW and IVF"
    url: https://www.youtube.com/watch?v=chz74Mtd1AA
related_formulas: [cosine-similarity, euclidean-norm]
---

## What problem the index solves

Finding the vectors closest to a query vector is conceptually trivial:
compare the query against every stored vector and keep the best matches. This
is **exact nearest-neighbour search**, it is always correct, and its cost
grows linearly with the size of the corpus. At ten thousand vectors nobody
notices the cost. At ten million, a single query comparing against every
vector becomes the dominant cost in your request's latency budget.
Approximate indexes exist to change the shape of this problem by organising
vectors so that most of them never need to be examined for a given query, at
the cost of occasionally missing a true nearest neighbour.

## HNSW, the intuition behind the graph

Think of a highway system feeding into arterial roads feeding into local
streets. Hierarchical Navigable Small World graphs build several layers of a
graph on exactly that shape: the top layer is sparse and long-range, and each
layer below gets progressively denser and more local.

A search starts at a random point in the sparse top layer and greedily moves
toward whichever neighbour is closest to the query. Once it can't improve any
further at that layer, it drops down to the next denser layer and repeats the
same greedy walk from where it left off, continuing until it reaches the
finest, most local layer. This reaches a good answer in a number of steps
that grows only logarithmically with corpus size rather than linearly — the
entire source of the speed gain. The cost: the greedy walk occasionally
commits to a direction that wasn't quite optimal, so it stops at a
good-but-not-best match.

## IVF, the intuition behind clustering

Inverted File indexing instead partitions the vector space into a fixed
number of clusters — found by a clustering algorithm run once at build
time — and assigns every stored vector to its nearest cluster centre. A query
only compares against the vectors in the handful of clusters whose centres
are nearest to the query itself, skipping every cluster deemed unlikely to
contain a good match entirely. The tunable parameter here, often called
`nprobe`, is how many of the nearest clusters to actually search: probing only
one cluster is fast and risks missing a true match that happened to land just
across a cluster boundary; probing many clusters approaches the accuracy of
exact search while giving back most of the speed advantage.

## Recall as the metric, and the dial that trades against it

**Recall at k** measures what fraction of the true top-k nearest neighbours an
approximate search actually returns, compared against what exact search would
have found. Both HNSW's traversal breadth parameter and IVF's `nprobe` are
direct dials on this trade-off: searching more broadly recovers recall closer
to exact search's ceiling at a real cost in latency and compute, and the right
setting is a measured choice for your corpus and your latency budget, not a
default left untouched.

## Quantization, for when memory is the bottleneck

Storing a full-precision vector for every item in a huge corpus can itself
become the memory bottleneck, independent of search speed. **Scalar
quantization** reduces each number in a vector from a 32-bit float to a
coarser 8-bit integer, cutting memory roughly fourfold at a small, usually
acceptable accuracy cost. **Product quantization** goes further, splitting
each vector into sub-vectors and representing each sub-vector by the nearest
entry in a small, shared codebook rather than storing it directly — a much
larger memory reduction, at a correspondingly larger accuracy cost that needs
measuring against your own recall requirements rather than assumed.

## Filtering, and why it interacts badly with approximate search

An approximate index finds candidates by proximity in the graph or clusters,
not by any filter you apply afterward. Filtering strictly after retrieval —
find the nearest hundred vectors, then discard any that do not match a
metadata condition — can leave far fewer usable results than requested if the
filter is selective, since the search never knew to look past the discarded
candidates for ones that would have matched the filter too. Properly
*filtered* search instead pushes the constraint into the traversal itself, so
the index only ever considers candidates that satisfy the filter while
searching, which most mature vector databases support but with real
differences in how well they implement it.

## When you do not need a dedicated system at all

A dedicated vector database is a genuine operational commitment: another
system to run, back up, monitor and keep synchronised with the source of
truth. A great many applications never outgrow `pgvector` running inside a
Postgres database they already operate, keeping vectors beside the rows they
describe and turning metadata filtering into an ordinary SQL join rather than
a special case. Scale actually measured against your real corpus size and
latency requirements is a reason to migrate to a dedicated system;
anticipation of scale that has not yet arrived usually is not.
