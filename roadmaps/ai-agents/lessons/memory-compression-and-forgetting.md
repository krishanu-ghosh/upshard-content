---
title: Memory compression and forgetting
description: Why an agent must deliberately summarize or discard old memory, and the concrete strategies for deciding what survives.
roadmap: ai-agents
node: memory-compression-and-forgetting
group: memory
video:
  provider: youtube
  id: "WP3hjUXd918"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 3780
level: intermediate
prerequisites: [vector-memory-and-rag-for-agents]
takeaways:
  - Unmanaged memory grows without bound as an agent accumulates episodes and retrieved facts, which eventually blows the context window, degrades the relevance of what's retrieved, and raises cost per call — compression and forgetting have to be deliberate choices, not accidents of running out of space.
  - Periodic summarization collapses older context into a shorter standing summary, trading some detail for a bounded size that still preserves the gist of what happened.
  - Importance-weighted retention keeps memories that are referenced often and drops ones that are not, using actual usage as the signal for what's worth keeping rather than age alone.
  - Aggressive summarization is a real tradeoff, not a free win — detail dropped today because it seemed unimportant can turn out to be exactly the detail a future task needed, and there's no way to know in advance which details those will be.
check_questions:
  - q: "Why can't an agent just keep every memory it ever stores, given that storage itself is cheap?"
    a: The cost isn't storage — it's that every memory an agent might retrieve has to compete for space in a fixed-size context window and adds to the cost of every call it's included in, and a retrieval step searching over a huge, undifferentiated pile of old memories gets noisier and less precise, so unbounded growth degrades both relevance and cost even when disk space is not the constraint.
  - q: "What's the core tradeoff in importance-weighted retention, and why doesn't it fully solve the forgetting problem?"
    a: It keeps memories that get referenced often and drops ones that don't, which is a reasonable proxy for usefulness — but a memory can be rarely referenced simply because the situation that would make it relevant hasn't come up yet, so retention by past usage can still discard something that turns out to matter the first time that situation finally does arise.
deeper: []
related_formulas: []
---

## Memory that only grows is not a feature

Every lesson in this group so far has been about adding memory: storing
episodes, embedding them, retrieving them by similarity. Left unchecked, that
accumulation becomes its own problem. Picture an agent that's been running
for months, storing an episode after every meaningful interaction. Three
things degrade at once. The context window can't hold everything relevant to
a long-running task, even after retrieval narrows it down. A vector search
over tens of thousands of stored memories returns noisier top matches than a
search over a few hundred well-curated ones. And every call that includes
retrieved memory pays for those extra tokens, whether or not they end up
mattering.

None of this is a bug in storage or retrieval — it's what happens when a
system is built to write memory but never to prune it. Compression and
forgetting are the missing other half: deciding, on purpose, what gets
condensed and what gets dropped, rather than discovering the limit by
hitting it.

## Periodic summarization: trading detail for a bounded size

The most direct strategy is **periodic summarization**: at some interval —
after every N conversations, or whenever stored context crosses a size
threshold — an agent runs a pass that collapses older, granular memories
into a shorter standing summary. Imagine a long-running project assistant
that has stored forty separate episodic memories about a single client
engagement over six months: individual notes about scope changes, specific
dates mentioned, preferences stated once. Once that count crosses a
threshold, a summarization pass might collapse those forty entries into one
paragraph: "Client engagement started in April, scope expanded twice (May,
August), prefers async updates over calls, current deadline is the 14th."
The agent now carries one dense paragraph instead of forty scattered
entries, at a fraction of the token cost, and future retrieval has one clear
summary to pull back instead of forty near-duplicate fragments to sort
through. The tradeoff is explicit: the exact wording and minor detail of
each of those forty original notes is gone, replaced by whatever the
summarization step judged worth keeping.

## Importance-weighted retention: let usage decide

A second strategy keeps the original memories intact longer but decides
selectively which ones survive, using **importance weighting**: track how
often each stored memory actually gets retrieved and used across sessions,
and keep the ones referenced frequently while letting rarely-used ones decay
or get pruned outright. This inverts the logic of summarization — instead of
compressing everything on a schedule, it lets actual demand determine value.
A fact that gets pulled into context in nearly every session (say, a user's
core working style) earns permanent retention; a fact that was stored once
and never retrieved again (a one-off remark from a single conversation) is a
candidate for deletion. This is closer to how caching systems evict
least-recently-used entries, applied to an agent's own memory store instead
of a cache.

## The real cost of getting this wrong

Both strategies share the same fundamental risk: you are making an
irreversible bet about what will matter later, using only what you know now.
Aggressive summarization is efficient exactly because it throws detail
away — and that is also its failure mode. The one sentence trimmed out of a
forty-entry summary because it looked minor at the time can be precisely the
detail a future task needs, with no way to recover it once the original
entries are gone. Importance weighting carries a mirrored version of the
same risk: a memory that has never been referenced isn't necessarily
unimportant. It may simply describe a situation that hasn't come up yet, and
pruning it on a low-usage signal means the agent has no answer the first
time that situation does arise.

## Treating forgetting as a design decision

The practical takeaway is that there is no compression strategy that is
free of this risk, which is exactly why it has to be a deliberate,
measured choice rather than something that happens by accident once a
context window fills up. A well-designed agent picks its summarization
cadence and its retention signal on purpose, matched to what the task
actually needs to never forget (a user's core preferences, a safety
constraint) versus what it can afford to compress or drop (the blow-by-blow
detail of how a long-resolved issue got fixed). Getting this wrong in either
direction is visible immediately: too little forgetting and the agent gets
slower, more expensive, and noisier in what it retrieves; too much
forgetting and it starts confidently missing details it genuinely used to
know.
