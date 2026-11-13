---
title: Building an MCP server
description: What it actually takes to write an MCP server that exposes a tool — a name, a schema, a function, and a process a client can connect to.
roadmap: ai-agents
node: building-an-mcp-server
group: model-context-protocol
video:
  provider: youtube
  id: "zX3pMXmRaPE"
  start: 0
  end: null
creator:
  name: Back To Code
  channel_url: https://www.youtube.com/@BackToCode
duration_seconds: 720
level: intermediate
prerequisites: [mcp-hosts-clients-and-servers]
takeaways:
  - An MCP server is an ordinary program that declares a set of tools, each with a name, a description, and an argument schema, and implements the function that runs when a client invokes one.
  - The description and schema are not documentation for a human reader — they are what the model sees when deciding whether and how to call the tool, so vague wording directly degrades how reliably the model uses it.
  - "Running the server is a separate step from defining the tools inside it: the server process listens for a client connection and dispatches incoming calls to the matching function by name."
check_questions:
  - q: Why does the wording of a tool's description matter as much as the code behind it?
    a: The model never sees the function's implementation — it only sees the name, description, and argument schema when deciding whether to call the tool and what arguments to pass, so a vague or ambiguous description leads directly to the model calling the tool incorrectly or not calling it when it should.
  - q: What two separate things does writing an MCP server actually require, beyond the business logic itself?
    a: It requires declaring each tool's interface — its name, description, and a schema for its arguments — so a client knows how to call it, and separately running a server process that listens for connections and routes each incoming call to the right function by name.
deeper:
  - label: "Building Your Own MCP Server is THIS EASY! (Python + FastMCP) — Thomas Janssen"
    url: https://www.youtube.com/watch?v=VWrJ14vLqJc
related_formulas: []
---

## From protocol to code

The previous lesson described the server as one of three roles in MCP's
architecture — the piece exposing tools, resources, or prompts over the
protocol. This lesson is about what writing one of those servers actually
looks like in practice. Stripped down, building an MCP server is three
pieces of work: declare each tool's interface, write the function behind it,
and run a process that a client can connect to. None of the three is
exotic — most of the work is ordinary application code — but each one has
a detail that trips people up the first time.

## Declaring a tool's interface

Every tool a server exposes needs three things attached to it: a **name**
the model will reference when it decides to call the tool, a **description**
explaining in plain language what the tool does and when to use it, and a
**schema** specifying what arguments it takes and their types. This triplet
is exactly what the model sees — not the function's source code, not
internal comments, just the name, description, and schema. If the
description says "looks something up" instead of "returns the current
weather for a given city, given a city name as a string," the model has
meaningfully less to work with when deciding whether this tool is the right
one to call for a given user request, and it will guess arguments less
reliably too.

## A worked example: one tool, defined

Consider a server that exposes a single tool, `get_weather`, which takes a
city name and returns a short text description of current conditions. Using
Python's MCP SDK with the FastMCP-style decorator pattern, the whole
definition is small:

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("weather-server")

@mcp.tool()
def get_weather(city: str) -> str:
    """Return current weather conditions for the given city."""
    # In a real server this would call a weather API.
    return f"It's 18°C and partly cloudy in {city}."

if __name__ == "__main__":
    mcp.run()
```

Three things are doing real work here. The `@mcp.tool()` decorator is what
registers `get_weather` as a callable tool rather than a plain Python
function — without it, the server never advertises this function to a
client at all. The type hint `city: str` and the docstring together become
the schema and description the model sees: the SDK inspects the function
signature to build the argument schema automatically, which is exactly why
getting the docstring and parameter names right matters as much as the
implementation. And `mcp.run()` is the piece that turns this file from a
script into a long-running server process, listening for a client to
connect and send it a call.

When a client sends a request to invoke `get_weather` with `city="Austin"`,
the server's job is mechanical from there: look up the registered function
by name, validate the incoming arguments against the schema, call the
function with those arguments, and send the return value back over the
connection as the tool's result. None of that dispatch logic is something
you write by hand in the example above — the SDK handles it — but it is
worth knowing it happens, because it's the reason a tool's declared schema
has to actually match what the function expects: a mismatch is caught at
the dispatch step, not discovered deep inside your own code.

## Running the server

Defining tools and running the server are genuinely separate concerns. A
file full of `@mcp.tool()` definitions does nothing on its own; `mcp.run()`
is what starts the process listening on whatever transport the server is
configured for — commonly standard input/output for a server launched
locally by a host, or an HTTP-based transport for one reachable over a
network. A client only sees a server once that process is running and
reachable, which is why a common first debugging step when "the model can't
see my tool" turns out to be nothing fancier than: is the server process
actually running, and is the client pointed at it correctly.

## Beyond one tool

A real server rarely stops at one tool. Scaling this pattern up mostly means
adding more `@mcp.tool()`-decorated functions to the same file — a
`get_forecast` alongside `get_weather`, say — each with its own name,
description, and schema, all served by the same running process and the
same client connection. The discipline that matters as the tool count grows
is keeping each tool's description specific and its schema accurate, because
a model choosing between five similarly-named tools is only as good as the
descriptions that distinguish them.
