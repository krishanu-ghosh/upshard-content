---
title: MCP hosts, clients, and servers
description: The three roles in MCP's architecture, and how a chat app, its built-in client, and a separate tool server fit together.
roadmap: ai-agents
node: mcp-hosts-clients-and-servers
group: model-context-protocol
video:
  provider: youtube
  id: "7KlapF6BfNU"
  start: 0
  end: null
creator:
  name: Learn with Manoj
  channel_url: https://www.youtube.com/@LearnwithManoj
duration_seconds: 420
level: intermediate
prerequisites: [what-is-mcp]
takeaways:
  - MCP architecture has three distinct roles — the host (the user-facing application), the client (the component inside the host that speaks MCP), and the server (the separate process exposing tools, resources, or prompts).
  - A host can run several clients at once, each client maintaining its own one-to-one connection to a single server, so one application can be wired to many tool servers simultaneously.
  - The server is a separate process from the host, often running locally or reachable over a network, which is what lets the same server be reused unmodified by a completely different host.
check_questions:
  - q: If a chat application is connected to both a GitHub MCP server and a filesystem MCP server at the same time, how many clients is it running, and why?
    a: It is running two clients, one per connection, because each client in MCP maintains a dedicated one-to-one link to exactly one server — a single client cannot multiplex two unrelated servers.
  - q: Why does it matter that the server is a separate process from the host rather than code embedded inside it?
    a: Because the server exists independently of any one application, the same server binary can be launched by a completely different host later — a GitHub server built for one chat app works unmodified inside an IDE or a different assistant, since both talk to it through the same protocol rather than through app-specific code.
deeper:
  - label: "How MCP servers work: Components, logic, and architecture — Vinh Nguyen"
    url: https://www.youtube.com/watch?v=m7CkdD-Mh_M
related_formulas: []
---

## Three roles, not two

It is tempting to think of MCP as just "an app talking to a tool," but the
protocol deliberately names three roles, and the distinction between the
first two is where most of the confusion starts. The **host** is the
application the person is actually using — Claude Desktop, an IDE, a custom
internal chat tool. The **client** is not a separate program the user
interacts with; it is a component living inside the host, a piece of code
whose only job is to speak the MCP protocol on the host's behalf. The
**server** is the thing on the other end: an independent process that
exposes a specific set of tools, resources, or prompts and waits for a
client to connect to it.

The reason to separate "host" from "client" at all, rather than just saying
"app talks to server," is that a host can embed more than one client, and
each client connection is deliberately narrow — one client, one server. That
narrowness turns out to be the key design choice that makes the rest of the
architecture simple to reason about.

## A worked example: Claude Desktop and a GitHub server

Picture Claude Desktop with MCP support enabled. Claude Desktop here is the
**host** — it's the window the person is typing into. Inside it, the
application has built in MCP client support: when the person configures a
connection to "the GitHub MCP server," Claude Desktop spins up a **client**
dedicated to that one connection. That client then launches or connects to
a separate program — the GitHub **MCP server** — which is a small piece of
software, maintained independently of Claude Desktop entirely, that knows
how to talk to the GitHub API and exposes a handful of capabilities over
MCP: a tool called something like `list_pull_requests`, another called
`create_issue`, maybe a resource exposing a given repository's README.

When the person asks "what pull requests do I have open right now," the
model decides to call `list_pull_requests`. That call doesn't go to GitHub
directly from the model — it goes from the host, through its client, over
the MCP connection, to the GitHub server, which is the only piece in this
chain that actually knows how to authenticate to GitHub and form the right
API request. The server runs the request, gets a list of pull requests back
from GitHub's API, and returns that list over the same MCP connection,
through the client, back into the host, where it becomes part of what the
model sees and responds with.

Now suppose the same person also wants the assistant to read files from a
local project folder. Claude Desktop, still the one host, now configures a
*second* client — a separate instance of the client logic — connected to a
filesystem MCP server, an entirely different small program whose only job is
exposing safe, scoped file operations. The host is now running two clients
side by side, each one tied to exactly one server, each server knowing
nothing about the other's existence. Nothing about adding the second
connection required touching the first one.

## Why the one-client-per-server rule matters

Keeping each client scoped to a single server connection is what lets
servers stay simple and independently reusable. A server never needs logic
for "which of my several connected clients is this request from" — it only
ever has one client on the other end of its connection, so its internal
state about that session is unambiguous. It also means a server author never
needs to know or care which host will eventually embed it: the GitHub server
in the example above works identically whether it's wired into Claude
Desktop, a different chat application, or an IDE's AI assistant feature,
because from the server's point of view it is just talking to *a* client
over MCP, and the protocol is the same regardless of which host that client
lives inside.

## The host's job versus the server's job

It's worth being precise about what each side is actually responsible for,
because the split is easy to blur. The host owns the user experience: the
conversation, the model calls, deciding when to invoke a tool, and rendering
the result. The host's embedded client owns only the mechanics of the MCP
connection itself — sending requests, receiving responses, handling the
protocol's handshake. The server owns the capability itself: the actual code
that lists pull requests, reads a file, or runs a query, plus whatever
credentials or local access that requires. None of these three pieces needs
to know how the other two are implemented internally — only that they agree
on the protocol connecting them. That separation is precisely what makes it
possible to swap in a different server, or wire the same server into a
different host, without rewriting anything on the other side.
