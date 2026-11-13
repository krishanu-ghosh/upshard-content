---
title: "MCP vs. custom integrations"
description: When MCP's standardization earns its overhead, and when a direct, custom tool integration is simply the better call.
roadmap: ai-agents
node: mcp-vs-custom-integrations
group: model-context-protocol
video:
  provider: youtube
  id: "_VXcDKI7JvA"
  start: 0
  end: null
creator:
  name: AWS Developers
  channel_url: https://www.youtube.com/@awsdevelopers
duration_seconds: 540
level: intermediate
prerequisites: [tool-definition-and-schemas, building-an-mcp-server]
takeaways:
  - A custom integration wires a tool directly into one application's code, with no protocol overhead and no extra process to run, but it only ever serves that one application.
  - An MCP server costs more upfront — a separate process, a protocol handshake, schema declarations — but that cost is paid once and the server then works with any MCP-compatible client, present or future.
  - "The deciding question is reuse: a tool one app will ever use is cheaper as a direct integration, while a tool multiple apps or agents across an organization need is cheaper, over time, as an MCP server."
check_questions:
  - q: Why can a custom integration be the technically better choice even though MCP is the newer, more general standard?
    a: A custom integration skips the protocol handshake, the separate server process, and the schema machinery entirely, calling the tool's logic directly in-process — for a tool only one application will ever use, that saved overhead is pure win with no offsetting reuse benefit, since there is no second client to share the standardization with.
  - q: What tips a tool from "write it as a custom integration" to "worth wrapping in MCP"?
    a: Reuse across more than one consumer — once a second application, agent, or team wants the same capability, a custom integration means maintaining duplicate or diverging copies of the same logic, while one MCP server serves every one of them through the same interface, which is what makes the standardization cost worth paying.
deeper:
  - label: "MCP vs API Explained: Do You Really Need MCP? — KodeKloud"
    url: https://www.youtube.com/watch?v=kn6dxL53NkM
related_formulas: []
---

## Two ways to connect a tool to a model

By this point in the roadmap you've seen two different ways to give a model
access to a capability. A **custom integration** is what the tool
definition and schema lesson walked through directly: you write a function,
describe it with a name and a JSON schema, hand that description to the
model, and when the model calls it, your application code runs the function
in-process and feeds the result back into the conversation. An **MCP
server** wraps that same kind of function behind a standard protocol, in a
separate process, reachable by any client that speaks MCP. Both approaches
end with the model successfully calling a tool. The question this lesson
answers is which one to reach for, because treating MCP as the default for
every tool is a real, common mistake.

## What a custom integration actually costs — and saves

A custom integration is about as cheap as tool-calling gets. There is no
second process to start, no protocol handshake, no schema translation layer
between "what I wrote" and "what MCP expects" — the function the model
calls is the function your application already has in memory. For a tool
that exactly one application will ever use, this is close to free: you
write the function once, describe it once, and that description lives right
next to the code that implements it. There is nothing to deploy separately,
nothing extra to monitor, and nothing extra that can go down independently
of the application itself.

The cost of this simplicity shows up the moment a second consumer appears.
If a different internal application wants the same capability, the custom
integration offers nothing to share — the second app's engineers either
import your function directly (coupling two codebases together) or rewrite
the same logic against their own application's integration pattern. Either
path tends to drift: a bug fix or an added parameter applied to one copy
quietly does not reach the other.

## What an MCP server costs — and buys

An MCP server pays real overhead upfront: you run a separate process,
implement the protocol's handshake and message format (or more realistically,
use an SDK that does it for you), and declare the tool's schema in the shape
MCP expects. None of that logic exists in a custom integration. What it buys
in exchange is that the server, once written, is reusable by construction:
any application with an MCP client — today's, or one built eighteen months
from now by a team that has never heard of the original project — can
connect to it and get the exact same tool with zero additional work on the
server side.

## A worked comparison

Take two concrete cases. First: an internal script that reformats a
specific report before a single dashboard app ingests it. Exactly one
application calls it, the format is specific to that dashboard, and no other
team has a plausible reason to want this exact transformation. Wrapping
this in MCP buys nothing — there is no second client to amortize the
protocol overhead across, so you'd be running an extra process and
maintaining a schema purely for ceremony. A custom integration, the
function called directly from the dashboard's own code, is strictly the
better engineering call here.

Second: a "query the company's analytics warehouse" tool that a data team
wants to expose. At the outset only one internal chatbot uses it. But within
the same quarter, three more teams want the same capability — a Slack bot,
an internal agent that drafts reports, and an IDE-integrated assistant used
by analysts. If this had been built as a custom integration inside the
original chatbot, each of those three new consumers would need its own
copy of the warehouse-querying logic, each one needing its own fixes when
the warehouse schema changes. Built once as an MCP server instead, all four
consumers — the original chatbot plus the three later ones — connect to the
same server, and a fix to how the warehouse is queried happens in exactly
one place and is immediately available to everyone.

## The actual decision rule

The pattern underneath both cases is reuse, not complexity or cleverness.
A tool used by exactly one application, with no realistic near-term reason
a second consumer will want it, is cheaper and simpler as a direct, custom
integration — skip MCP's overhead entirely. A tool you expect multiple
applications or agents across an organization to need, now or soon, is
worth the upfront cost of an MCP server, because that cost is paid once
while the reuse benefit compounds with every additional consumer. When
you're unsure which bucket a given tool falls into, start with the custom
integration — it's cheap to build and cheap to discard. Migrate to an MCP
server only once a second real consumer actually shows up, rather than
speculatively building the more general version for a reuse case that may
never materialize.
