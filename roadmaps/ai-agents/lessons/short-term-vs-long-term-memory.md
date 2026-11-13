---
title: "Short-term vs. long-term memory"
description: What fits in an agent's context for the current task versus what gets deliberately saved to an external store for next time.
roadmap: ai-agents
node: short-term-vs-long-term-memory
group: memory
video:
  provider: youtube
  id: "W2HVdB4Jbjs"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1020
level: intermediate
prerequisites: [the-agent-loop]
takeaways:
  - Short-term memory is whatever fits in an agent's context window for the current task — it vanishes when the session ends unless something explicitly saves it.
  - Long-term memory is information deliberately written to an external store — a database, a file, a vector index — and deliberately read back into context in a future session.
  - "The two solve different problems: short-term memory tracks what the agent has done so far in this task, while long-term memory tracks what it should remember about this user or project across tasks."
  - Treating everything as short-term memory forces an agent to relearn the same facts every session; treating everything as long-term memory bloats every prompt with details most tasks don't need.
check_questions:
  - q: What happens to an agent's short-term memory when its session ends, and why?
    a: It disappears, because short-term memory is just the contents of the context window for that run — once the process exits or the conversation is reset, nothing keeps that text around unless the agent explicitly wrote it somewhere durable first.
  - q: Why can't an agent just keep everything in long-term memory and skip the short-term/long-term distinction?
    a: Long-term memory has to be retrieved and re-inserted into the context window before it's usable, and every token retrieved competes for the same limited window as the actual task — so an agent that pulls in everything it has ever stored would blow its context budget and bury the current task under irrelevant history.
deeper:
  - label: "Memory Harnesses for Long-Running Research Agents — Stefania Druga, Sakana.ai — AI Engineer"
    url: https://www.youtube.com/watch?v=R3-anFK1YM8
related_formulas: []
---

## The context window is a scratchpad, not a filing cabinet

An agent's context window — the text it can actually attend to at a given
moment — behaves like a scratchpad. Everything the agent writes there, reads
there, or reasons over while working through a task is **short-term memory**:
the current file it's editing, the last few tool outputs, the plan it
sketched three turns ago. It is cheap to use because it's already right
there in the prompt, but it is fragile. Close the session, start a new one,
or simply run long enough that old turns get truncated out of the window,
and that information is gone. Nothing about the scratchpad itself persists.

**Long-term memory** is the opposite trade: something durable instead of
something free. It lives outside the context window entirely — in a
database row, a JSON file, a vector index — and has to be deliberately
written to and deliberately read back. The "deliberately" matters twice.
Nothing lands in long-term memory by accident; some part of the system has
to decide a fact is worth persisting and issue a write. And nothing comes
back out of long-term memory by accident either; some part of the system has
to decide a fact is relevant right now and issue a read that pulls it back
into the context window, where it becomes short-term memory again for the
duration of that session.

## A worked example: a coding assistant across two sessions

Picture a coding assistant partway through refactoring a function. Its
short-term memory right now holds the open file, the diff it's about to
apply, the test output from the last run, and the plan it formed a few turns
ago for how to split the function. None of that is written anywhere
permanent — it exists because it's sitting in the context window for this
task, and if the session crashed right now, all of it would be lost except
whatever had already been saved to disk as actual file changes.

Now suppose this same user has told the assistant, across several past
sessions, that they prefer early returns over nested conditionals, and that
they want test files updated in the same commit as the code they test. A
well-designed assistant doesn't rediscover these preferences by re-reading
old transcripts every time — it stored them, once, in a long-term memory
store as a short profile: "prefers early returns," "always pairs tests with
the change." At the start of each new session, before any work begins, that
profile gets pulled back into the context window. The refactor the assistant
proposes in session three uses the same style the user corrected it toward
in session one, without the user repeating themselves. The function body and
diff are short-term; the standing preference is long-term, and the handoff
between them — write once, read at the start of every future session — is
the entire mechanism.

## Why conflating the two breaks an agent

An agent that only has short-term memory forgets the user the moment the
session ends: every conversation starts from zero, no matter how much
context was built up before. An agent that treats everything as long-term
memory has a different failure mode — it tries to persist and re-inject
every detail of every session, and the context window fills up with stale or
irrelevant history before the model even gets to the actual question being
asked right now. Good memory design is deciding, deliberately, what belongs
in which bucket: transient task state stays short-term and is allowed to
evaporate, while durable facts about the user, the project, or the world get
promoted to long-term storage and pulled back in only when relevant.

## Where this goes next

This split is the foundation the rest of this group builds on. Once you
accept that long-term memory means something was deliberately written and
must be deliberately retrieved, the next questions are mechanical: what
exactly gets written (a raw transcript, or a distilled fact?), and how does
an agent find the right memory to retrieve out of everything it has ever
stored? Those are the subjects of the next two lessons — episodic memory and
vector retrieval — and both are really just more detailed answers to the
write-and-read problem introduced here.
