---
title: Working with Claude Code
description: A hands-on orientation to Claude Code's terminal-based agentic workflow, how it reads a codebase, and its permissions model.
roadmap: ai-product-builder
node: working-with-claude-code
group: building-with-ai-coding-agents
video:
  provider: youtube
  id: "ntDIxaeo3Wg"
  start: 0
  end: null
creator:
  name: Tech With Tim
  channel_url: https://www.youtube.com/@TechWithTim
duration_seconds: 2148
level: intermediate
prerequisites:
  - ai-coding-agents-overview
takeaways:
  - Claude Code works from a terminal session rooted in your project directory, and it builds its understanding of the codebase by reading files and running commands, the same way a new engineer would.
  - A project-level instructions file lets you hand it durable context once — conventions, commands, architecture notes — instead of repeating it in every prompt.
  - Its permissions model gates risky actions (editing files, running shell commands, reaching the network) behind approval, with settings to pre-allow safe, repeated operations.
  - Directing it well means giving a goal and constraints, not dictating every line, then reviewing the diff it proposes rather than the words it used to describe it.
  - Because it runs real commands, Claude Code can verify its own work — run the test suite, check a build, read an error — which closes the loop that a chat-only assistant can't.
check_questions:
  - q: How does Claude Code figure out how a codebase is organized if nobody explains it up front?
    a: It reads files directly — listing directories, opening source files, searching for patterns — building its own model of the project the way a new contributor would, and a project instructions file can shortcut this by stating conventions and architecture once.
  - q: What's the purpose of Claude Code's permissions model?
    a: It puts a checkpoint in front of actions with real consequences, like editing a file or running a shell command, so you approve them before they happen; routine, low-risk actions can be pre-approved so review effort concentrates on changes that matter.
  - q: Why is reviewing the diff more important than reading the agent's explanation of what it did?
    a: The explanation is the agent's summary of its own work and can be wrong, incomplete, or glossing over a side effect; the diff is the actual change that will ship, so it's the only reliable source for judging correctness.
deeper:
  - label: "A Complete Guide to Claude Code - Here are ALL the Best Strategies"
    url: "https://www.youtube.com/watch?v=amEUIuBKwvg"
related_formulas: []
---

## A terminal session, not a chat window

Claude Code runs as a session in your terminal, rooted at a project directory you point it at. That placement matters: it isn't a chat window with a code snippet pasted in, and it isn't limited to one open file the way an editor extension often is. From the terminal it can see the whole project tree, open any file in it, run the project's own build and test commands, and use the output of one command to decide the next. The workflow looks less like "ask a question, get an answer" and more like handing a task to a capable collaborator who happens to work by typing into the same shell you do.

## How it learns a codebase

Point Claude Code at an unfamiliar repository and it starts the way any new engineer would: it reads the directory structure, opens files relevant to the task, searches for where a function or a route is defined, and checks how existing code handles a similar case before writing new code of its own. That's slower than a human skimming a mental map they already have, but it's also how an agent avoids inventing a different convention than the one your codebase already uses. You can speed this up by maintaining a short project-level instructions file — the house style, the commands to run tests, the architectural decisions worth knowing — so the agent doesn't have to rediscover them from scratch on every session.

## Directing it

The most useful instructions describe the destination and the constraints, not the path. "Add rate limiting to the login endpoint, reuse the existing middleware pattern in `middleware/`, and don't touch the session logic" gives Claude Code a clear goal, a convention to follow, and a boundary. A vague instruction like "make login more secure" forces it to guess at scope, which is where agents either over-deliver (touching files you didn't want changed) or under-deliver (missing the thing you actually meant). Being specific about what's in scope and out of scope does more for the quality of the result than any amount of phrasing cleverness.

## The permissions model

Because Claude Code can edit files and run commands, it needs a boundary around what it does without asking first. Its permissions model puts a checkpoint in front of consequential actions — writing to a file, running a shell command, reaching out over the network — so you can see and approve what's about to happen before it happens. You can pre-approve specific, repeated, low-risk actions (running a known test command, for instance) so you aren't interrupted for things you've already decided are fine, while anything novel or broader in scope still stops and asks. This is the practical version of the autonomy dial from the previous lesson: the permissions model is where you set it.

## Closing the loop by actually running things

The part of this workflow that a plain chat assistant can't offer is verification. Claude Code doesn't just propose code and hope it's right — it can run the test suite it just touched, execute the script it just wrote, or read the error a failed build produced, and then fix what it finds. That loop of edit, run, observe, and adjust is what lets it catch a typo or a wrong import before you ever see the diff. It doesn't make the output infallible — you still review everything before it ships — but it does mean a meaningful share of the obvious mistakes get caught before they reach you at all.
