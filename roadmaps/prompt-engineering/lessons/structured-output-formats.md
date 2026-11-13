---
title: Structured output formats
description: Schema-constrained decoding, JSON mode, and the prompt-parse-retry fallback — three ways to get data back instead of prose.
roadmap: prompt-engineering
node: structured-output-formats
group: output-control
video:
  provider: youtube
  id: "NGEZsqEUpC0"
  start: 0
  end: null
creator:
  name: Lucas Soares (Automata Learning Lab)
  channel_url: https://www.youtube.com/@automatalearninglab
duration_seconds: 784
level: beginner
prerequisites: [few-shot-prompting]
takeaways:
  - Schema-constrained decoding masks invalid tokens during generation, so the model is structurally unable to produce output that violates your schema.
  - JSON mode only guarantees syntactically valid JSON — it says nothing about which keys or types that JSON actually contains.
  - The prompt-parse-retry fallback works with any model but is the least reliable of the three, so it needs a retry loop built in from the start.
check_questions:
  - q: Why is schema-constrained decoding more reliable than JSON mode?
    a: It constrains the model's token probabilities at every generation step so invalid tokens can never be produced, while JSON mode only guarantees the output is syntactically valid JSON without checking it against any particular schema.
  - q: When is the prompt-parse-retry approach still necessary?
    a: When the model or provider offers neither schema-constrained decoding nor JSON mode, which is common with models you don't control the inference stack for.
deeper:
  - label: "AI Anytime — Gemini LLM JSON Mode: Generate Structured Output from LLM"
    url: https://www.youtube.com/watch?v=zzKFWwVEQEM
  - label: "Leon van Zyl — LangChain Output Parsers: Get Structured JSON from Any LLM"
    url: https://www.youtube.com/watch?v=vvZ4cyxl99Q
related_formulas: []
---

Ask a model for "a JSON object with a name and age" and you'll usually get something
that looks like JSON. Usually. It might wrap the object in a markdown code fence, add a
sentence before it ("Sure, here's the JSON:"), use a key you didn't ask for, or emit a
number as a quoted string. None of that breaks a human reading it. All of it breaks a
`JSON.parse()` call downstream. Getting a model's output into a shape your code can
consume reliably is "structured output," and the problem gets harder exactly where it
matters most: at scale, across edge cases, over months of a provider changing things
under you.

There are three real approaches, and they differ a lot in how hard they try to
*guarantee* correctness rather than just increase the odds of it.

## Schema-constrained decoding: the model cannot emit invalid tokens

The strongest guarantee comes from constraining generation itself. You hand the API a
JSON Schema — field names, types, which fields are required — and the provider's
inference stack walks the model's token probabilities at every step, masking out any
token that would violate the schema. The model isn't being asked nicely to follow a
shape; it is structurally incapable of producing output that doesn't validate. OpenAI's
Structured Outputs (`response_format: {type: "json_schema", ...}`), Gemini's
`response_schema`, and most self-hosted serving stacks via grammar-constrained decoding
(outlines, guidance) all work this way. Where your provider supports it, this is the
default choice — it turns "the model should return valid JSON matching my schema" into
"the model will return valid JSON matching my schema," full stop.

## JSON mode: valid syntax, not validated structure

A weaker, older guarantee is "JSON mode" — a flag that tells the model to emit
syntactically valid JSON, with no enforcement of what keys or types that JSON actually
contains. You'll never get back malformed JSON, but you can still get `{"name": "Alex"}`
when you asked for `{"name": string, "age": number}`, because the mode only constrains
syntax, not schema. It's a real improvement over nothing, but if your provider offers
schema-constrained decoding, prefer that — JSON mode is what you reach for when the
model doesn't support anything stronger.

## The universal fallback: prompt, parse, retry

Neither of the above exists for every model or provider, and plenty of production
systems still run against models with no structured-output mode at all. The fallback
that works everywhere: describe the exact shape you want in the prompt itself (often
with a literal example), parse whatever comes back, and if parsing fails, feed the parse
error back to the model and ask it to correct its own output. A minimal version looks
like: "Return only a JSON object matching `{\"name\": string, \"age\": number}`. No other
text." Then parse the reply, and if `JSON.parse` throws, send the error back with "That
failed to parse: `<error>`. Return only valid JSON this time." Two or three retries
catches nearly everything a first attempt misses. This is the least reliable of the
three — the model is free to ignore your formatting instructions, wrap the JSON in
prose, or drift from the schema under its own judgment — but it's also the only option
that works with a model you don't control the inference stack for, which is most of them
from the outside. Frameworks like LangChain's output parsers automate the retry loop;
the mechanism underneath is always these same three steps.

## Picking one

Default to schema-constrained decoding whenever your provider offers it — it's strictly
better than the alternatives and costs nothing extra to use. Fall back to JSON mode when
schema constraints aren't available but syntactic JSON is. Reserve the prompt-and-parse
approach for models or providers that support neither, and build the retry loop in from
the start rather than discovering in production that one in every few hundred responses
doesn't parse. Whichever you use, keep your schema as small and flat as you can — nested,
optional, and union-typed fields are where even schema-constrained decoding starts
producing technically-valid-but-wrong answers, like an empty array where you expected
one item, or a fallback value in an optional field the model didn't know how to fill.
