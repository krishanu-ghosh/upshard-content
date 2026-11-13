---
title: Context engineering
description: Deciding what goes in the window and why — write, select, compress, isolate — and the failure modes of a window packed to the limit.
roadmap: ai-engineer
node: context-engineering
group: prompting
video:
  provider: youtube
  id: "4GiqzUHD5AA"
  start: 0
  end: null
creator:
  name: LangChain
  channel_url: https://www.youtube.com/@LangChain
duration_seconds: 1326
level: intermediate
prerequisites: [prompt-anatomy, the-context-window]
takeaways:
  - Context engineering is the discipline of deciding what occupies the window on each call, and it subsumes prompt wording as one part of a larger problem.
  - The four moves are writing context outside the window, selecting what to bring back, compressing what must stay, and isolating what belongs to a different task.
  - Adding relevant material can still make results worse, because every token added competes with every other for the model's attention.
check_questions:
  - q: Why can adding a relevant document to the context make an answer worse?
    a: Attention is finite and shared. A new document dilutes everything else, can contradict another source, and pushes earlier material into positions the model weights less heavily.
  - q: What does it mean to write context outside the window?
    a: Persisting state — notes, plans, intermediate results — to a file or store rather than carrying it in the conversation, then selecting back only the part a given step needs.
deeper:
  - label: "Chroma via Hamel Husain — Context Rot, when long context fails"
    url: https://www.youtube.com/watch?v=3s_N60u0jEY
  - label: "AWS Developers — Four context engineering techniques for agents"
    url: https://www.youtube.com/watch?v=-q_u6nrVdgo
related_formulas: []
---

## From wording to contents

Prompt engineering asks how to phrase an instruction. Context engineering
asks what should be in the window at all: which documents, how much history,
which tool definitions, which prior results, and what gets dropped to make
room.

Think of the context window as a desk. Prompt engineering is choosing your
words carefully. Context engineering is deciding what's actually sitting on
the desk — which files are open in front of you and which are back in the
cabinet — because a desk buried under every document you might conceivably
need is harder to work from than one with just the three you're using right
now.

For anything long-running, the second question dominates. A single-turn
call is a prompt. An agent working for twenty minutes across fifty tool
calls is a context-management problem with a prompt somewhere inside it.

## The four moves, each with concrete technique

**Write** means persisting state somewhere other than the conversation
window — a scratchpad file the agent can append to, a running task list, a
small key-value store — so information survives without permanently
occupying context. A coding agent that writes its plan to a file on disk,
then re-reads only the relevant section later, is using this move.

**Select** means retrieving only what the current step actually needs,
instead of carrying everything that might ever be relevant. This is
retrieval applied to the agent's own working memory, not just to an external
document store: choosing which three of fifty available tools to expose for
one particular step, or pulling back only the one earlier message relevant
to the question just asked, is selection.

**Compress** means summarizing history instead of carrying it verbatim —
replacing the full text of the last fifteen tool calls with a three-sentence
summary of what was tried and what was learned, once that detail stops
earning its place in the budget. Compression is lossy by design, which makes
deciding what to lose a real engineering decision, not an automatic default.
Summarize away the specific error message that caused a retry, and the agent
can repeat the same mistake, because the detail that would have prevented it
is exactly what got compressed away.

**Isolate** means giving a sub-task its own clean context window so its
intermediate mess never pollutes the orchestrating agent's context — the
mechanism behind
[multi-agent systems](/roadmaps/ai-engineer/multi-agent-systems), where a
sub-agent can spend an entire window exploring a dead end and report back
only its conclusion.

## Context rot, with the specific finding

Research measuring model performance as input length grows — see this
lesson's deeper link — finds that accuracy degrades non-uniformly. It's not
a smooth, gentle slope. Certain kinds of added content, particularly
irrelevant or distracting material placed among the relevant facts, hurt
disproportionately more than an equal volume of simply longer but relevant
text.

A document that duplicates information already present, or a retrieved
passage that's topically related but doesn't actually answer the question,
can measurably drag down accuracy on the real answer sitting nearby in the
same window. This is a direct argument for the re-ranking step in
[similarity search](/roadmaps/ai-engineer/similarity-search): a few precisely
relevant passages reliably beat many loosely relevant ones.

## When to compact, as a concrete heuristic

A workable rule many production agents use: trigger compaction once the
context fills to somewhere around seventy percent of the window, rather than
waiting until it's full. Wait until the last moment, and compaction has to be
aggressive and rushed, discarding detail indiscriminately. Start earlier, and
you get a more careful summary that preserves what later steps are likely to
need.

Re-stating the original goal near the end of the context on every turn, even
after heavy compaction, is a small, cheap habit that keeps a long-running
agent from drifting away from what it was actually asked to do — the
"losing the plot" failure covered in
[the agent loop](/roadmaps/ai-engineer/the-agent-loop).

## The discipline is mostly subtractive

The question worth asking repeatedly isn't what else the model might find
useful, but what's currently in the window that isn't earning its tokens.
Nearly every context-engineering failure in production is an excess, not a
deficiency — too much retrieved, too much history carried, too many tools
offered at once. The fix is almost always removal, not addition.
