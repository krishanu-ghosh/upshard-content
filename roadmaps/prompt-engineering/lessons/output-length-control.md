---
title: Output length control
description: Why asking for "a short answer" and setting max_tokens are not the same control, and what happens when a response hits the limit mid-thought.
roadmap: prompt-engineering
node: output-length-control
group: output-control
video:
  provider: youtube
  id: "aOm75o2Z5-o"
  start: 0
  end: null
creator:
  name: AssemblyAI
  channel_url: https://www.youtube.com/@AssemblyAI
duration_seconds: 840
level: beginner
prerequisites: [structured-output-formats]
takeaways:
  - A prompt-level length instruction lets the model finish its thought within a shape; max_tokens cuts generation off regardless of whether the thought is finished.
  - A response that ends mid-sentence or mid-JSON-object is the signature of hitting a max_tokens ceiling, not the model choosing to be brief.
  - Where a provider exposes a verbosity parameter, it controls how expansive the model chooses to be and is more reliable than a prompted "be concise" instruction.
check_questions:
  - q: What's the difference between asking a model to "be concise" and setting max_tokens?
    a: Asking for conciseness lets the model finish its thought within a shorter response; max_tokens forcibly stops generation at a token count regardless of whether the response is complete.
  - q: What's a reliable sign that a response was cut off by max_tokens rather than finished naturally?
    a: The response ends abruptly mid-sentence, mid-word, or mid-structure, like an incomplete JSON object, rather than reaching a natural stopping point.
deeper: []
related_formulas: []
---

"Keep it short" is the most common and least reliable length instruction you can give a
model. It works often enough to feel trustworthy and fails often enough — on a long
enough input, a complex enough question, a model having a verbose day — to be a bad
default for anything you're shipping. Controlling output length actually means reaching
for one of a few distinct mechanisms, and they fail in different ways, so picking the
wrong one for the job produces a different kind of bad output than you expected.

## Prompt-level length control: ask, and be specific

The weakest but most flexible control is asking in the prompt itself, and specificity
matters enormously here. "Be concise" is vague enough that a model will interpret it
differently depending on what else is in the prompt; "respond in exactly three bullet
points, each under 15 words" gives the model a concrete target it can actually hit.
Giving a format that implies a length — "a one-paragraph summary," "a single sentence,"
"a table with one row per item" — tends to work better than an adjective, because it
describes the shape of the output rather than asking the model to self-judge what
"short" means for this particular answer. This is the only mechanism that can make a
response *shorter and more complete* at once: the model still finishes its thought, just
within whatever shape you asked for.

## The hard cutoff: max output tokens

`max_tokens` (or `max_output_tokens`, depending on provider) is a different mechanism
entirely: it tells the API to stop generating after N tokens, full stop, regardless of
whether the model was mid-sentence. This is not the model choosing to be brief — it is
the API cutting the model off. A response that hits this limit ends exactly where the
token count ran out, which can be mid-word, mid-JSON-object, or mid-reasoning-step. Use
`max_tokens` as a safety ceiling against runaway generation and cost, not as your primary
length-control mechanism — if you actually want a short, *complete* answer, combine it
with a prompt-level instruction that gets the model to naturally finish within that
budget, rather than relying on the cutoff to do the work.

## The newer knob: verbosity

Some providers now expose a `verbosity` parameter, separate from both of the above: a
dial (typically low/medium/high, or a numeric scale) that asks the model to be more or
less expansive in how it explains itself and formats its response, independent of any
specific length target. This differs from a prompt instruction in that it's a first-class
API parameter rather than text the model has to interpret and honor alongside everything
else in the prompt, and it differs from `max_tokens` in that it changes *how much the
model chooses to say*, not where generation gets forcibly cut off. Where available, it's
worth using over a prompted "be verbose" or "be concise" instruction, since it's handled
by the model's own training for that behavior rather than parsed out of your prompt text.

## Combining them without truncating mid-thought

The practical pattern: set `max_tokens` generously — high enough that a normal, complete
response never hits it — and use prompt-level instructions (or `verbosity`, where it
exists) to get the length and shape you actually want. If you need a hard guarantee that
a response will never exceed a strict length for cost or latency reasons, set
`max_tokens` tightly but pair it with an explicit prompt instruction asking for a
response that fits comfortably under that count, so the model aims to finish early
rather than getting cut off at the wire. A response that ends mid-sentence is a reliable
tell that `max_tokens` did the trimming rather than the model choosing to stop; if you
see that in production, the fix is almost always to raise the token ceiling and tighten
the prompt instruction instead.
