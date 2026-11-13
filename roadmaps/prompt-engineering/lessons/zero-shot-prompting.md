---
title: Zero-shot prompting
description: Asking a model to do a task from a plain instruction alone, with no worked examples, and knowing when that is not enough.
roadmap: prompt-engineering
node: zero-shot-prompting
group: basic-techniques
video:
  provider: youtube
  id: "cpNzKLLFQ6s"
  start: 0
  end: null
creator:
  name: Alfa Yohannis
  channel_url: https://www.youtube.com/@alfayohannis
duration_seconds: 144
level: beginner
prerequisites: [temperature-and-randomness]
takeaways:
  - A zero-shot prompt gives the model a task description and nothing else — no worked examples, no demonstrated pattern to copy.
  - It works best on tasks the model has clearly seen many times during training, where the instruction alone is enough to pick the right pattern.
  - It breaks down on unusual output formats, ambiguous instructions, or domain-specific conventions the model cannot infer from the wording alone.
check_questions:
  - q: What makes a prompt "zero-shot" rather than "few-shot"?
    a: It contains no worked examples of the task being done, only an instruction and optional context, with the model expected to infer the right response format from the wording alone.
  - q: Why does zero-shot prompting tend to fail on unusual output formats?
    a: Nothing in the prompt shows the model what the output should look like, so it falls back on whatever format is statistically typical for similar instructions, which may not match what you actually want.
deeper: []
related_formulas: []
---

## One instruction, nothing else

Zero-shot prompting means giving a model an instruction and nothing else — no
example inputs, no example outputs, no demonstrated pattern to copy. The model
has to infer, purely from what it learned during training, what a correct
response looks like. "Zero" refers to the number of worked examples in the
prompt, not to the amount of context: a zero-shot prompt can still include
background information, constraints, or a persona. It just never shows the
model what a finished, correct answer actually looks like.

A concrete zero-shot prompt:

> Classify the sentiment of this review as positive, negative, or neutral.
> Review: "The battery life is fine, but the screen cracked after a week."
> Sentiment:

No example of a correctly classified review appears anywhere. The model has
to recognize "classify sentiment" as a familiar task shape and infer, from the
instruction and the review text alone, which of the three labels applies and
in what format to return it.

## Why it works at all

Modern instruction-tuned models have seen enormous numbers of task
descriptions paired with correct completions during training and
instruction-tuning, so a request phrased in a conventional way often maps
cleanly onto a pattern the model already knows how to execute. Sentiment
classification, summarization, translation, simple extraction, and rewriting
are common enough that zero-shot usually gets you most of the way there with
a capable model. This is also why zero-shot is the right place to start:
it costs the fewest tokens, takes the least prompt-engineering effort, and if
accuracy is already good enough, every example you would add to a few-shot
prompt is pure context-budget spent for no gain.

## Where zero-shot breaks down

Three failure modes show up repeatedly.

**Unusual output formats.** If you want the sentiment label returned as a
single lowercase word with no punctuation, say so explicitly. A prompt that
just says "classify the sentiment" without specifying the output shape gets
back whatever format is statistically typical for sentiment-classification
answers in the training data — a full sentence, a JSON object, or a label
with inconsistent casing from one call to the next.

**Ambiguous task framing.** "Summarize this" is ambiguous along several axes
the model has to guess at: how long should the summary be, should it preserve
technical jargon or simplify it, is the reader someone who already read the
source or someone deciding whether to? Each guess is a chance to guess wrong,
and with no example anchoring the prompt, nothing corrects a bad one.

**Domain-specific or house-style conventions.** A model has no way to know
that your team labels tickets "P0/P1/P2" rather than "urgent/high/low," or
that your style guide bans the Oxford comma, unless you tell it — and even
then, an instruction like "use our house style" without specifying what that
means gives the model nothing concrete to follow.

## Writing a zero-shot prompt that holds up

The fix for all three failure modes is the same: be as explicit as possible
about the output you want, even though you are not demonstrating it. State
the exact format ("respond with one word: positive, negative, or neutral"),
resolve ambiguity up front (length, audience, tone), and spell out any
house-specific conventions rather than assuming the model will infer them. A
well-specified zero-shot prompt often outperforms a sloppy few-shot one,
because explicit instructions remove guesswork that implicit examples
sometimes leave only partly resolved.

The moment a task's output shape is easier to show than to describe in
words — a particular table layout, a specific tone, a format with several
interacting rules — that is the signal to stop refining the instruction and
add a worked example instead. That is exactly where few-shot prompting, next,
picks up.
