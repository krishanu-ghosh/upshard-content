---
title: Tool definitions and schemas
description: A tool is only as useful as its description — the name, prose, and JSON schema that tell a model when to call it and how to fill in its arguments.
roadmap: ai-agents
node: tool-definition-and-schemas
group: tools
video:
  provider: youtube
  id: "zuMw0pkPXpU"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 1518
level: intermediate
prerequisites: [what-are-tools]
takeaways:
  - A tool definition has three parts — a name, a natural-language description, and a JSON-schema-style parameter spec — and the model only ever sees that definition, never your function's actual source code.
  - "The description is doing most of the real work: it's the only thing telling the model when the tool applies and how its arguments map onto the user's request, so vague wording produces wrong or missing calls even when the schema is technically correct."
  - Constraining a parameter's type, enum, or format in the schema prevents a whole class of errors before the model ever gets a chance to guess wrong.
check_questions:
  - q: What three pieces make up a tool definition, and which one does the most work in getting the model to use the tool correctly?
    a: A name, a natural-language description, and a JSON-schema-style parameter specification; the description does the most work, because it's the model's only source of information about when the tool applies and what its arguments mean.
  - q: Why does constraining a parameter with an enum or a type, rather than leaving it as a free-text string, reduce errors?
    a: A free-text string lets the model phrase a value any way it likes, including ways your code can't parse, while an enum or a strict type rules out those ill-formed values before the call is ever made, catching the error at definition time instead of at execution time.
deeper: []
related_formulas: []
---

## From "a function" to "something a model can use"

A tool, from the previous lesson, is a function your code can execute. But the model
calling it never sees that function's source code — it has no idea what's inside
`get_weather` or how `search_flights` queries a database. All it has is a tool
*definition*: a structured description the model reads instead of the implementation.
That definition does one job — telling the model, in terms it can reason about, when
this tool is relevant and what values it needs — and if that job is done badly, the
model either picks the wrong tool, picks the right tool with wrong arguments, or
doesn't realize a tool applies at all. Writing good tool definitions is a real skill,
closer to writing clear API documentation for a human developer than to programming.

## The three parts of a definition

Every tool definition, across OpenAI, Anthropic, and Gemini's APIs, boils down to the
same three fields, even though the exact wrapping syntax differs slightly between
providers.

- **Name** — a short, unambiguous identifier like `search_flights`, not `tool1` or
  `handleRequest`. The model uses the name itself as a signal, so a descriptive name
  does real work before the description is even read.
- **Description** — a sentence or two of plain English explaining what the tool does
  and, often more usefully, when to use it and when not to. This is read by the model
  the same way a paragraph of your prompt is: as natural language to reason over, not
  as code.
- **Parameters** — a JSON-schema-style object naming each argument, its type, whether
  it's required, and any constraints on its allowed values.

Here is a concrete definition for a flight-search tool:

```json
{
  "name": "search_flights",
  "description": "Search for available flights between two airports on a given date. Use this when the user asks about flight options, prices, or availability — not for booking or cancelling an existing flight.",
  "parameters": {
    "type": "object",
    "properties": {
      "origin": {
        "type": "string",
        "description": "Three-letter IATA airport code, e.g. 'JFK'."
      },
      "destination": {
        "type": "string",
        "description": "Three-letter IATA airport code, e.g. 'LIS'."
      },
      "date": {
        "type": "string",
        "format": "date",
        "description": "Departure date in YYYY-MM-DD format."
      },
      "cabin_class": {
        "type": "string",
        "enum": ["economy", "premium_economy", "business", "first"],
        "description": "Defaults to economy if the user does not specify one."
      }
    },
    "required": ["origin", "destination", "date"]
  }
}
```

## Why the description carries so much weight

Notice what the `search_flights` description does beyond stating the obvious. It
doesn't just say "searches for flights" — it draws a boundary: *use this for options
and prices, not for booking or cancelling*. That boundary matters enormously the
moment your agent has more than one travel-related tool. Without it, a model asked
"can you cancel my flight to Lisbon?" might reasonably guess that `search_flights` is
the closest available match and call it, returning a list of flights for a cancellation
request — a wrong tool call that looks superficially plausible. A vague description
like "flight tool" invites exactly this kind of confusion; a description that states
both purpose and boundary heads it off.

The same principle applies at the parameter level. `cabin_class` says what happens if
the user doesn't mention a class at all — defaults to economy — which stops the model
from either omitting the field unpredictably or inventing a guess. `date`'s format
hint tells the model the exact string shape your backend expects, rather than letting
it emit "next Tuesday" or "12th of March" and leaving your parser to cope.

## What makes a description ambiguous

A description is ambiguous when it leaves the model to guess something you actually
know and could have stated. "Searches flights" doesn't say what counts as a valid
airport identifier, whether round trips are supported, or whether price is always
included. Each of those gaps is a place where two reasonable models — or the same
model on two different days — could fill in the blank differently. The fix isn't to
write a longer description for its own sake; it's to write down the specific
distinctions that would otherwise be ambiguous: what the tool is for, what it is not
for, what each parameter means, and what happens when an optional one is left out.

## Schemas catch what prose can't

Natural-language description constrains behavior by persuasion — the model reads it
and tries to comply, but nothing stops it from writing `cabin_class: "fancy"` if the
prose alone said "a travel class." The JSON schema constrains behavior structurally:
an `enum` of exactly four allowed values makes `"fancy"` invalid before your code ever
runs, and a `required` list means a call missing `origin` can be rejected immediately
rather than failing deep inside your flight-search logic with a confusing error. Good
tool design treats these two layers as complementary — description for *why* and
*when*, schema for *what shape* — rather than leaning on one to do the other's job.
