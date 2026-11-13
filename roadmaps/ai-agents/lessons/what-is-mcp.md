---
title: "What is the Model Context Protocol?"
description: "Anthropic's open standard for wiring LLM apps to tools and data — it turns N times M integrations into N plus M."
roadmap: ai-agents
node: what-is-mcp
group: model-context-protocol
video:
  provider: youtube
  id: "T4-BdZQUCSE"
  start: 0
  end: null
creator:
  name: PyTorch Foundation
  channel_url: https://www.youtube.com/@PyTorch
duration_seconds: 1980
level: intermediate
prerequisites: [native-function-calling]
takeaways:
  - MCP is an open protocol, released by Anthropic in November 2024, that standardizes how LLM applications connect to external tools, data sources, and prompts.
  - Without a shared standard, connecting every app to every tool needs a custom integration per pair — the N×M problem — and that count grows multiplicatively as either side grows.
  - "MCP collapses N×M integrations into N+M: each application builds one MCP client, and each tool or data source is wrapped in one MCP server, and any client can then talk to any server."
  - The protocol defines three building blocks a server can expose — tools the model can call, resources it can read, and prompts it can reuse — over a common transport.
check_questions:
  - q: What specific problem does the N×M framing describe, and what makes it worse than it sounds?
    a: It describes needing a custom integration for every pairing of an app and a tool, so the integration count is the product of how many apps and how many tools exist, not their sum — meaning every new app or every new tool multiplies the work rather than adding a fixed amount to it.
  - q: Why does wrapping a tool in an MCP server pay off across multiple applications but not for a single one-off script?
    a: An MCP server is built once and exposes the tool over a standard interface that any MCP-compatible client can connect to, so the cost of building it is paid once and reused by every app that adopts it; a script used by exactly one app never recoups that extra layer, since there is no second client to share it with.
deeper:
  - label: "Why we built—and donated—the Model Context Protocol (MCP) — Anthropic"
    url: https://www.youtube.com/watch?v=PLyCki2K0Lg
  - label: "MCP Explained in 7 Minutes — CodeWithMasood"
    url: https://www.youtube.com/watch?v=84B0dPlcvE0
related_formulas: []
---

## The problem before MCP had a name

By late 2024, most LLM applications that called external tools did so through
bespoke, hand-rolled glue code. A chat app that wanted to search the web, query a
database, and read files from disk wrote three separate integrations, each
shaped around that specific tool's API, its own auth scheme, and its own way of
describing what it does. None of that code was reusable anywhere else. If a
second application wanted the same three capabilities, it wrote the same three
integrations again from scratch, because the first app's glue code was wired
directly into its own internals.

This is the N×M problem, and it is worth sitting with the arithmetic. Suppose
there are 10 LLM applications in active use — IDEs, chat assistants, internal
agents — and 20 tools worth connecting to them: GitHub, Slack, a filesystem,
a company's internal database, and so on. Full coverage, every app wired to
every tool, requires 10 × 20 = 200 separate integrations. Add one more
popular tool and the number jumps by 10, not by 1, because every existing app
now needs its own new integration to reach it. Add one more app and the
number jumps by 20. The cost of growth compounds in both directions at once,
and nobody's individual integration work helps anybody else.

## How MCP reframes the problem

MCP, Anthropic's open protocol released in November 2024, does not try to
make any single integration easier to write. Instead it standardizes the
*interface* between the app side and the tool side, so that the two can be
built independently and still work together. Concretely, it splits the
problem into two fixed roles:

- A **client** lives inside the application (the host) and speaks MCP on
  the app's behalf. An application author writes this once.
- A **server** wraps a tool, a data source, or a set of prompts and exposes
  them over MCP in a standard shape. A tool author — or the tool's own
  maintainers — writes this once.

Any MCP client can talk to any MCP server, because both sides agree on the
same message format and the same categories of thing a server can offer.
Go back to the 10 apps and 20 tools: instead of 200 bespoke integrations,
you need 10 clients and 20 servers — 30 pieces of integration work, each
built once and then reused by everyone on the other side. That is the N+M
shape: the ten app authors build a client each, the twenty tool authors
build a server each, and the connections between them come for free because
they share a protocol. Adding an eleventh app now costs one new client, not
twenty new integrations; adding a twenty-first tool costs one new server,
not ten.

## What a server actually exposes

MCP is specific about what a server can offer across the protocol, and it
breaks into three categories. **Tools** are functions the model can decide
to call — "list my open pull requests," "run this SQL query" — each with a
name, a description, and a schema for its arguments, much like the function
definitions covered in the native function calling lesson. **Resources** are
pieces of data a client can read, such as a file's contents or a database
record, without the model necessarily invoking a function call to get them.
**Prompts** are reusable, parameterized prompt templates the server offers,
so that a well-crafted prompt for a specific task does not need to be
re-invented inside every application that wants to use it. A single server
can expose any mix of the three; a minimal one might expose just a handful
of tools.

## Why this matters even for a single team

The N×M argument is easiest to see across an entire ecosystem, but it applies
inside one organization too. A team that builds an internal "query the
analytics warehouse" tool as a one-off function bolted onto a single chat
app has quietly created an N×M problem in miniature: the moment a second
internal tool wants the same capability, someone copies the glue code and
starts maintaining two divergent copies. Wrapping that same capability in
one MCP server, built once, means every current and future internal
application that speaks MCP gets it for free, and a change to how the
warehouse is queried needs to happen in exactly one place.
