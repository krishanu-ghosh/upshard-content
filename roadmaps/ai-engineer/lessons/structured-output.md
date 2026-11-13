---
title: Structured output
description: Getting schema-conforming JSON out of a model reliably, using constrained decoding rather than asking politely and hoping.
roadmap: ai-engineer
node: structured-output
group: working-with-models
video:
  provider: youtube
  id: "xpvFinvqRCA"
  start: 0
  end: null
creator:
  name: Efficient NLP
  channel_url: https://www.youtube.com/@EfficientNLP
duration_seconds: 1040
level: intermediate
prerequisites: [calling-a-model-api]
takeaways:
  - Asking a model for JSON in the prompt is a request; constraining the decoder is a guarantee, because invalid tokens are never sampled in the first place.
  - Constrained decoding works by masking the scores of every token that could not continue a valid document, given a grammar or state machine.
  - Structural validity is not semantic correctness — a schema-conforming object can still contain invented values, so validation and evaluation remain necessary.
check_questions:
  - q: How does constrained decoding guarantee valid JSON?
    a: At each step a grammar or state machine determines which tokens could legally continue the document, and every other token's score is set to negative infinity before sampling. An invalid token is unreachable.
  - q: If the output always conforms to the schema, what is left to check?
    a: Whether the values are right. Constrained decoding enforces shape, not truth; a required field will be filled even when the source text does not support any answer.
deeper:
  - label: "BugBytes — Instructor and Pydantic for structured LLM output"
    url: https://www.youtube.com/watch?v=3xUW1Do9zOs
related_formulas: [softmax]
---

## Why prompting for JSON is not enough

"Respond only with valid JSON" works most of the time, and most of the time is
the problem. At scale a small failure rate is a steady trickle of parse
errors, and they cluster exactly where you least want them: long outputs,
unusual inputs, anything near the token ceiling. A prompt instruction competes
with every other pressure on the distribution — it is a request, not a
constraint, and the model remains free to sample a token that breaks it.

## The mechanism: masking before sampling

Think of it like a bouncer checking IDs before anyone reaches the dance
floor, rather than a sign on the wall asking people to behave once inside. At
each decoding step, a grammar or a finite-state machine built from your JSON
schema tracks exactly which continuations are legal given everything
generated so far. Immediately after an opening `{`, only a string key or a
closing `}` is legal. Immediately after a `:`, only a value of the declared
type is legal. Every token in the vocabulary that would produce an illegal
continuation has its logit driven to negative infinity before the softmax
runs, so its probability becomes exactly zero. The model still chooses freely
among whatever remains legal — it genuinely decides which key to fill in what
order, within the bounds the grammar allows — but it can't sample a token
that would break the schema, because that token was never a candidate to
begin with. That's the core difference between structured output as a
guarantee and a prompt instruction as a request: one operates before
sampling and makes violation impossible, the other operates by persuasion
and can fail.

A complication worth knowing: most models tokenize multiple characters per
token, so a single token might represent `","` or `":{` — several grammar
transitions worth of text at once. Building the mask correctly means
reasoning about which multi-character tokens are legal given the grammar
state, not simply which single characters are, which is most of why this
machinery is nontrivial to implement from scratch and is usually provided by
a library or the model provider rather than hand-rolled.

## JSON mode versus full schema enforcement

Two tiers are common. **JSON mode** guarantees only that the output
syntactically parses as JSON — some object, any shape — which prevents
garbled output but does not guarantee any particular field exists or has the
right type. **Schema-guided decoding** goes further, enforcing an entire JSON
Schema or an equivalent type definition, so a `price` field is guaranteed to
be a number and a `status` field is guaranteed to be one of an enumerated set
of strings. The schema-guided tier is strictly more useful whenever the
application needs to treat the result as a typed object rather than just
parseable text.

## Function-calling-style output as a special case

[Tool calling](/roadmaps/ai-engineer/tool-calling) uses exactly this same
constrained-decoding machinery under the hood: a tool's parameter schema is a
JSON Schema, and the model's "I want to call this tool with these arguments"
output is schema-guided generation wearing a different name. Recognising this
means the design advice for a good tool schema — clear types, enums instead of
free strings, required fields genuinely required — is the same advice for any
structured-output schema, not a separate skill.

## What it still does not buy you

Structural validity, and nothing else. A model asked for a required field will
produce one whether or not the source text actually supports an answer —
constraining the decoder makes a confident invention well-formed rather than
preventing the invention. Make fields nullable or optional wherever the
correct answer may genuinely be absent, so the model has a legal way to say
nothing, and validate the populated values on the way out using a library
like Pydantic or Zod, retrying the call with the validation error fed back in
if a value fails a business-logic check the schema could not express (a date
in the past, a total that does not match its line items). Keep failing cases
in your eval set — shape is checked by the decoder, meaning is checked by
[evals](/roadmaps/ai-engineer/why-evals-matter).
