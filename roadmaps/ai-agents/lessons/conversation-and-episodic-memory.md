---
title: Conversation and episodic memory
description: The difference between keeping a chat transcript around and extracting specific, reusable facts from past conversations.
roadmap: ai-agents
node: conversation-and-episodic-memory
group: memory
video:
  provider: youtube
  id: "R0OdB-p-ns4"
  start: 0
  end: null
creator:
  name: DeepLearningAI
  channel_url: https://www.youtube.com/@DeepLearningAI
duration_seconds: 1080
level: intermediate
prerequisites: [short-term-vs-long-term-memory]
takeaways:
  - Conversation memory is keeping recent turns of a chat around so replies stay coherent with what was just said — it's short-term by default and usually just a buffer of raw text.
  - Episodic memory is narrower and more deliberate — it stores discrete past events as reusable facts, so an agent can recall a specific prior episode and act on it without re-reading the whole transcript it came from.
  - A raw transcript answers "what was said"; an extracted episode answers "what happened, and what should I do differently because of it" — the second requires actually processing the conversation, not just storing it.
  - Turning a transcript into episodic memory means deciding what's worth keeping, which is a real design choice with a real cost if done badly — keep too little and useful facts get lost, keep everything and retrieval gets noisy.
check_questions:
  - q: "How does episodic memory differ from just keeping a conversation transcript around?"
    a: A transcript is the raw sequence of turns, useful only if you re-read it; episodic memory is the product of someone (or some process) deciding which specific events in that transcript are worth extracting as standalone, reusable facts, so the agent can recall "last time you preferred window seats" without searching through an entire past conversation to find it.
  - q: "Why can't an agent just keep appending every conversation turn to context forever instead of building episodic memory?"
    a: The context window has a fixed size, so an ever-growing transcript eventually has to be cut, truncated, or summarized anyway; episodic memory does that distillation deliberately and in advance, extracting the specific facts that matter so they survive even after the original conversation they came from is gone.
deeper:
  - label: "LangChain - Conversations with Memory (explanation & code walkthrough) — Sam Witteveen"
    url: https://www.youtube.com/watch?v=X550Zbz_ROE
related_formulas: []
---

## Conversation memory: staying coherent within a session

The simplest form of memory an agent needs is just enough of the recent
back-and-forth to keep a conversation coherent. If a user says "make it
shorter" on turn five, the agent needs turns one through four in view to
know what "it" refers to. This is **conversation memory**: a rolling buffer
of recent turns, usually just appended to the context window as plain text,
sometimes trimmed or summarized once it gets too long. It is short-term
memory in its most literal form — there is no extraction, no judgment about
what matters, just the raw record of what was said, kept around because the
current turn depends on it.

Conversation memory is necessary but shallow. It answers "what did we just
say to each other," and nothing more. It doesn't help the agent six weeks
from now, in a brand new session, with no access to this transcript at all.

## Episodic memory: extracting an event, not storing a transcript

**Episodic memory** is a different and more deliberate idea: instead of
keeping the whole conversation, extract specific, discrete events from it —
"episodes" — and store each one as a standalone fact the agent can retrieve
later, independent of the conversation it came from. The term borrows from
human cognitive science, where episodic memory is your memory of specific
events ("what I had for dinner last Tuesday") as opposed to semantic memory
(general facts you know, detached from when you learned them). For an agent,
an episode is typically a short, self-contained record: what happened, when,
and what matters about it going forward.

The worked example from the brief makes the distinction concrete. Suppose a
user asks a travel-booking agent to book a flight, and during that
conversation mentions they prefer a window seat. A system with only
conversation memory has this preference sitting somewhere in a transcript
from three weeks ago — technically present, practically unreachable, because
nothing is re-reading old transcripts before every new task. A system with
episodic memory instead extracted a fact at the time it happened: `{event:
"booked flight", preference: "window seat", date: ...}`. The next time this
user asks the agent to book a flight, that specific episode gets retrieved
and injected into context — not the entire prior conversation, just the one
fact that's relevant — and the agent books a window seat without being
asked again.

## The extraction step is where the real work happens

The hard part of episodic memory isn't storage — writing a record to a
database is trivial. The hard part is deciding, as a conversation unfolds,
*what counts as an episode worth keeping*. Extract too aggressively and you
store noise: every passing remark becomes a "fact," and retrieval later
returns clutter instead of signal. Extract too conservatively and you miss
exactly the detail that turns out to matter, like a seating preference
mentioned once, in passing, that never gets flagged as significant. In
practice this extraction is usually done by prompting a model to summarize a
conversation into candidate facts after it ends, then applying some filter —
recency, explicit confirmation, repetition across multiple conversations —
before committing anything to long-term storage.

## Why this sits between short-term memory and retrieval

This lesson sits directly between the short-term/long-term split from the
previous lesson and the retrieval mechanics coming next. Episodic memory is
one concrete answer to "what gets written" when you decide to persist
something long-term: not a transcript, but a distilled, timestamped event.
But deciding what to write is only half the problem — once you've stored
hundreds of episodes, you need a way to find the one that's relevant to the
task in front of you right now, out of everything you've ever stored. That
retrieval problem, solved with embeddings and similarity search, is exactly
where the next lesson picks up.
