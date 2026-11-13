---
title: Few-shot prompting
description: Zero-shot against few-shot, how to choose and order examples, and the cases where examples make the output worse.
roadmap: ai-engineer
node: few-shot-prompting
group: prompting
video:
  provider: youtube
  id: "ojtbHUqw1LA"
  start: 0
  end: null
creator:
  name: Elvis Saravia
  channel_url: https://www.youtube.com/@elvissaravia
duration_seconds: 643
level: beginner
prerequisites: [prompt-anatomy]
takeaways:
  - Examples teach form far more reliably than they teach judgment, so they help most where the output shape matters and least where the decision is subtle.
  - Examples are the clearest way to specify a format, and often replace several paragraphs of description.
  - A biased or unrepresentative example set gets learned just as faithfully as a good one — mistakes included.
check_questions:
  - q: When is adding examples likely to make results worse?
    a: When the examples aren't representative of real inputs, when they all share an incidental pattern the model copies, or when the task needs reasoning that the examples short-circuit by showing only final answers.
  - q: Why do examples cost more than an equivalent instruction?
    a: They occupy context on every call. That's a permanent tax on tokens and latency — which is exactly what makes a cached prefix worth arranging.
deeper:
  - label: "Arize AI — Few-shot prompting as an engineering decision"
    url: https://www.youtube.com/watch?v=ggXckjI_w-w
related_formulas: []
---

## Telling versus showing

An instruction tells the model what you want. An example shows it. Showing
usually wins, and the reason why tells you a lot about what examples are
actually good for.

If you want a new hire to format a pull request description your way, you
could write three paragraphs of style guidance — or you could hand them three
good PRs and say "like this." The three PRs communicate the shape instantly.
What they don't communicate is *why* those PRs made the calls they made.
That's the whole trade-off with few-shot prompting, compressed into one
analogy.

## What's actually happening mechanically

A model given examples inside a single prompt is doing **in-context
learning**: adjusting its behavior for that one call based on patterns in the
prompt, with no weight update involved at all. The leading theory is that
models develop something like induction circuits during training — internal
machinery that notices "this bit of input matched that pattern before, with
this output" and extrapolates the same relationship forward.

You don't need the mechanism to use the result. What matters is this:
examples get read as a pattern to continue, not as a rule to reason about.
That's why they're so strong for form and comparatively weak for judgment.
Show three correctly classified borderline cases, and the model doesn't learn
your classification rule — it learns three points, and draws its own boundary
through them. That boundary might not be the one you had in mind.

## Zero-shot, one-shot, few-shot, many-shot

**Zero-shot** means no examples, just the instruction. **One-shot** gives a
single example — often enough to pin down a format. **Few-shot**, the usual
case, gives somewhere between two and ten. **Many-shot**, made practical by
bigger context windows, gives dozens or hundreds, and performance on some
tasks keeps climbing well past where few-shot gains flatten out — at a steep
cost in tokens per call.

Where you land on that spectrum is a cost trade-off. A simple classification
task might need zero examples. A task with a genuinely unusual, hard-to-write
output convention can be worth many.

## Choosing and ordering your examples

Cover the real range of inputs, including the awkward ones — the edge case,
the ambiguous input, the one where the right answer is "not enough
information to decide." A set made only of clean, easy cases teaches the
model that clean, easy cases are all that exist. Hand it a genuinely
ambiguous input later, and it will force out a confident answer rather than
flag the ambiguity, because nothing in the examples ever showed that option.

Order matters too, and not in a way you'd predict. Models are measurably
sensitive to the sequence examples appear in — the last one tends to carry
outsized weight on what comes next. If your examples happen to be sorted by
something incidental, like every positive case pulled first because that's
how they sat in a spreadsheet, the model may read that incidental ordering as
part of the pattern itself. Shuffle the order and check whether the output
changes. It's a cheap test, and if the answer does change, that instability is
worth turning into an eval case of its own.

## Dynamic few-shot, and a genuine surprise

A more advanced setup retrieves the most relevant examples for each new input
at runtime, pulled from a larger bank, instead of reusing the same fixed
handful every time. Mechanically it's the retrieval from
[similarity search](/roadmaps/ai-engineer/similarity-search), just applied to
examples instead of documents. It tends to beat a static set once your input
distribution is varied enough that no fixed handful covers it well.

Here's the counterintuitive part: on some tasks, adding few-shot examples has
been shown to *hurt* the strongest models — a pattern researchers call
inverse scaling. A sufficiently capable model sometimes already follows a
clear instruction correctly on its own, and the examples end up anchoring it
toward an incidental pattern in them rather than the instruction's actual
intent. The lesson isn't "avoid examples." It's "test with and without them on
your own eval set," rather than assuming more guidance is always better.
