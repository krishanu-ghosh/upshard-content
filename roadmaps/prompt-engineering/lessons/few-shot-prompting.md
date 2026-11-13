---
title: Few-shot prompting
description: Showing a model worked examples of a task instead of just describing it, and knowing how many examples actually help.
roadmap: prompt-engineering
node: few-shot-prompting
group: basic-techniques
video:
  provider: youtube
  id: "Ns7oxTn5U6A"
  start: 0
  end: null
creator:
  name: Sundeep Saradhi Kanthety
  channel_url: https://www.youtube.com/@sundeepsaradhi
duration_seconds: 776
level: beginner
prerequisites: [tokens-and-context, zero-shot-prompting]
takeaways:
  - Few-shot prompting demonstrates the task with worked input/output pairs instead of only describing it, which pins down a format an instruction alone can leave ambiguous.
  - Examples cost context tokens, so each one is a trade-off against the budget covered in tokens-and-context, not a free way to improve accuracy.
  - Diverse, well-ordered examples generalize; narrow or repetitive ones teach the model to overfit to incidental details of the examples rather than the actual task.
check_questions:
  - q: What problem does adding examples solve that a clearer instruction usually cannot?
    a: Examples pin down an output format or edge-case behavior that is easy to show but hard to fully specify in words, closing gaps a purely verbal instruction leaves open to the model's own interpretation.
  - q: Why can adding more examples make a prompt worse, not better?
    a: Each example spends context budget and risks teaching the model incidental patterns from the examples themselves, such as a coincidental length or phrasing, rather than the underlying rule you actually want it to generalize.
deeper: []
related_formulas: []
---

## Showing instead of telling

Few-shot prompting means including a small number of worked examples —
input paired with the correct output — before asking the model to handle a
new, real input. Where a zero-shot prompt only describes the task, a few-shot
prompt demonstrates it, and demonstration pins down details that an
instruction in plain language tends to leave ambiguous.

Take a zero-shot prompt that under-specifies its output format:

> Extract the product name and price from this listing.
> "Wireless Mouse - Ergonomic Design - $24.99 - Free Shipping"

The model might return a sentence, a bulleted list, or a JSON object — all
reasonable readings of "extract," none of them guaranteed to match what you
actually need downstream. Adding two examples removes the guesswork:

> Listing: "USB-C Cable 6ft Braided - $8.50"
> Output: {"product": "USB-C Cable 6ft Braided", "price": 8.50}
>
> Listing: "Mechanical Keyboard RGB Backlit - $89.99 - Ships Tomorrow"
> Output: {"product": "Mechanical Keyboard RGB Backlit", "price": 89.99}
>
> Listing: "Wireless Mouse - Ergonomic Design - $24.99 - Free Shipping"
> Output:

Now the model has seen the exact key names, the number type for price, and
that shipping details get dropped — none of which the instruction spelled
out, and all of which the examples make unambiguous.

## Examples are not free

Every example in a prompt is tokens that count against the context budget
covered in the previous lesson. A handful of short examples is cheap; a dozen
long ones for a complex task can dwarf the actual input you're trying to
process, especially once you're paying per token or running close to a
model's context limit. The practical question is never "would one more
example help a little," since it almost always would — it's whether that
marginal token spend is worth it compared to the cost of the mistake you're
trying to prevent. For a high-volume, low-stakes classification task, two
or three tight examples usually beat ten exhaustive ones.

## More examples, worse results

Accuracy on most tasks improves sharply from zero examples to a handful, then
flattens — and can decline again past that point, for two concrete reasons.
First, each additional example after the first several adds diminishing
information: once the pattern is clear, more repetitions mostly just restate
it. Second, and more subtly, a narrow or repetitive example set teaches the
model incidental regularities that happen to be true of your examples but
aren't the actual rule. If all three worked examples in a sentiment-labeling
prompt happen to be one sentence long, the model may quietly learn "answers
are about one sentence" rather than "classify sentiment," and then struggle
the moment a real input is a full paragraph.

## Choosing examples deliberately

Three choices do most of the work. **Diversity**: cover the range of inputs
you expect, including awkward edge cases, not just the easy, central ones —
if ambiguous or negative cases matter in production, show at least one.
**Order**: models can be sensitive to the sequence examples appear in, and
putting the example most similar to the real input last, immediately before
the actual query, tends to anchor the response more reliably than burying it
first. **Count**: start with two or three and add more only if you can point
to a specific failure the additional example fixes, not as a reflexive
hedge against uncertainty.

## Where few-shot still isn't enough

Few-shot pins down *what* a correct answer looks like, but it does nothing
by itself to help a model work through a problem that requires several
reasoning steps to get right — a multi-step calculation or a chain of
dependent inferences can have the right output format demonstrated perfectly
in every example and still come out wrong, because the examples show the
destination, not the reasoning that gets there. That gap is exactly what
chain-of-thought prompting, next, is built to close.
