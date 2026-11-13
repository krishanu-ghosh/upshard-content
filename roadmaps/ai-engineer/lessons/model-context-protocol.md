---
title: Model Context Protocol
description: The problem MCP solves, how hosts, clients and servers fit together, and what a standard interface changes about wiring an agent to the world.
roadmap: ai-engineer
node: model-context-protocol
group: agents
video:
  provider: youtube
  id: "CQywdSdi5iA"
  start: 0
  end: null
creator:
  name: Anthropic
  channel_url: https://www.youtube.com/@anthropic-ai
duration_seconds: 1175
level: intermediate
prerequisites: [tool-calling]
takeaways:
  - MCP standardizes the connection between an application and a tool provider, so one integration works with any client that speaks the protocol.
  - A server exposes tools, resources and prompts. A client inside a host application connects to it and relays what it offers to the model.
  - A standard interface doesn't make a connection safe — an MCP server is code you're trusting with whatever the agent can reach.
check_questions:
  - q: What problem does MCP solve that tool calling alone does not?
    a: Tool calling defines how a model requests an action within one application. MCP defines how applications and tool providers connect at all, so one integration serves many clients instead of being rewritten per host.
  - q: Why does the distinction between tools, resources and prompts matter?
    a: They differ in who initiates. The model invokes a tool, the application reads a resource as context, and the user chooses a prompt. Collapsing them hides who's in control of each interaction.
deeper:
  - label: "Mahesh Murag (Anthropic) — Building agents with MCP, full workshop"
    url: https://www.youtube.com/watch?v=kQmXtrmQ5Zg
  - label: "David Soria Parra — The future of MCP"
    url: https://www.youtube.com/watch?v=v3Fr2JR47KA
related_formulas: []
---

## The integration problem

Before a standard existed, connecting a model to an external system meant
writing that integration once, for one specific host application. Your issue
tracker, your internal database, your file system — each connector got
rewritten separately for every application that wanted to use it, and every
host invented its own shape for what a "tool" even looked like.

MCP is the fix: a protocol so a tool provider gets implemented once, and any
compliant application can connect to it. Like USB-C replacing a drawer full
of proprietary chargers, the value here is almost entirely the network
effect of a shared standard, not any particular cleverness in the protocol's
own design.

## JSON-RPC underneath, and how a connection starts

Mechanically, MCP messages are JSON-RPC — a simple request-response format
where each message names a method and carries parameters, with every request
expecting a matching response correlated by an id.

A connection begins with an explicit **initialization handshake**: the
client states which protocol version and capabilities it supports, the
server replies with its own, and only once both sides have agreed on what's
available does either side send a request that actually does something. This
negotiation is what lets the protocol evolve without breaking older clients
talking to newer servers, or vice versa — each side only uses features the
other has explicitly confirmed it understands.

## The pieces, and who talks to whom

A **host** is the application the user is actually in — a chat interface, an
IDE, a custom agent. Inside that host, a **client** maintains a connection to
exactly one **server**: a separate process, local or remote, exposing some
set of capabilities. One host typically runs several clients side by side,
each connected to a different server — a filesystem server, a database
server, a ticketing server — and the host aggregates what all of them offer
into one combined view for the model.

## The three primitives, and who initiates each

**Tools** are model-invoked: functions the model may decide to call during
its own reasoning, exactly as covered in
[tool calling](/roadmaps/ai-engineer/tool-calling). **Resources** are
application-controlled: data the host itself decides to read and place into
context, like the currently open file in an editor, without the model asking
for it. **Prompts** are user-invoked: templates a person deliberately picks
from a menu, like a one-click "summarize this document," rather than
something the model or the application decided on its own.

A fourth, less-discussed primitive, **sampling**, flips the usual direction
entirely: it lets a server ask the client's own connected model to generate
something on the server's behalf — useful when a server needs a bit of model
reasoning as part of its own logic but has no model access of its own.

This distinction — who's in control of each interaction — is the part most
casual summaries of MCP flatten into "it's like tool calling." That misses
the host-controlled and user-controlled halves entirely.

## What's moved since the reference material was recorded

The transport layer has changed since MCP's original release. The initial
spec paired a simple standard-input/output transport for local servers with
a server-sent-events transport for remote ones. The remote transport has
since been reworked into a more capable streaming design, and authentication
for remote servers has moved toward a proper OAuth-based flow rather than the
simpler, less standardized schemes some early servers used. The reference
workshop in this lesson's deeper link is still the most complete walkthrough
of MCP's concepts — it's also the material most affected by this change, so
read its transport-specific sections as history, and check the current spec
for anything you're about to implement.

## A standard interface does not mean a trusted one

MCP makes connecting easy and does nothing about trust. A server can do
anything its own host process is capable of doing, and a server you didn't
write is free to describe its own tools however it likes — including
descriptions deliberately crafted to steer the model's behavior. That's
[prompt injection](/roadmaps/ai-engineer/prompt-injection) arriving through
the tool catalog instead of the user's message.

This is sometimes called a confused-deputy problem: the model, acting with
whatever privileges the host has granted it, gets manipulated by a malicious
server into misusing those privileges on the server's behalf instead of the
user's. Run third-party MCP servers with exactly the suspicion you'd apply to
any other dependency with network access and your credentials — read what it
can reach and what it can do before connecting it to anything that matters.
