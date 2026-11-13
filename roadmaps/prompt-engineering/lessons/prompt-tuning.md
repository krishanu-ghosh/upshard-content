---
title: Prompt tuning
description: A parameter-efficient training technique that learns a soft prompt instead of writing one — not a synonym for prompt engineering.
roadmap: prompt-engineering
node: prompt-tuning
group: optimization
video:
  provider: youtube
  id: "yu27PWzJI_Y"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 513
level: intermediate
prerequisites: [prompt-debiasing]
takeaways:
  - Prompt tuning freezes the base model's weights and trains a small set of continuous soft-prompt embeddings instead of writing natural-language text.
  - It needs a labeled task dataset and gradient-descent training, which makes it a machine-learning technique rather than a manual prompting skill.
  - It sits between manual prompt engineering and full fine-tuning on the compute-and-data spectrum, alongside prefix-tuning and LoRA.
check_questions:
  - q: What does "soft" mean in a soft prompt, and why can't you read one?
    a: It is a sequence of continuous embedding vectors learned by gradient descent, not tokens from the vocabulary, so it has no corresponding words to print or read — only numbers in embedding space.
  - q: Why is prompt tuning not just "automated prompt engineering"?
    a: Prompt engineering edits discrete, human-readable text aimed at a frozen model with no training step. Prompt tuning trains new parameters against a dataset and a loss function, the same mechanics used to train the model itself, just on a far smaller parameter set.
deeper: []
related_formulas: []
---

## Two different things share the word "prompt"

Every earlier lesson in this roadmap has been about writing better natural-language
text: choosing words, giving examples, structuring instructions. All of that is prompt
**engineering** — a human skill applied to a frozen model, with no training involved.
Prompt **tuning** is a different thing entirely, despite the name. It is a
parameter-efficient fine-tuning (PEFT) technique: a way to adapt a large language model
to a specific task by training a small number of new parameters, while leaving the
model's own weights untouched.

The confusion is understandable, because the two produce something that looks similar
at inference time — a sequence prepended to your real input, steering the model's
output. But the mechanism and the cost structure are nothing alike. One is something
you type. The other is something you train.

## What actually gets trained

A soft prompt is a short sequence of continuous embedding vectors — typically 10 to 100
of them — prepended to the input's token embeddings before they enter the model. These
vectors do not correspond to any real token in the model's vocabulary; you cannot
decode them back into words, because they were never words to begin with. They start
as random (or heuristically initialized) numbers and are updated by ordinary gradient
descent, exactly the way a model's own weights are updated during training, except here
every weight in the base model is frozen and only the soft-prompt embeddings receive
gradients.

Training requires a labeled dataset for the target task — input examples paired with
the outputs you want — and a loss function comparing the model's predictions against
those labels, backpropagated only into the soft prompt. After enough steps, the learned
embeddings reliably push the frozen model toward the desired behavior for that specific
task. A new task means training a new soft prompt from scratch; the technique produces
one small, task-specific artifact per task rather than one adapted model.

## Why bother, if you already have prompt engineering

Manual prompting has a ceiling. Natural language is a surprisingly lossy way to specify
exact, repeatable behavior — a human-written instruction can always be phrased a
different way, and models drift in how literally they follow phrasing across versions.
Prompt tuning searches a much larger, continuous space than any human would try by
hand, and for a narrow, well-defined task it can reliably outperform the best
discrete prompt anyone writes, because gradient descent, not trial and error, is doing
the search. It also decouples the task adaptation from the prompt text shown to an end
user, if any — the soft prompt lives behind the scenes.

The cost is what it has always been with any trained artifact: you need labeled data,
a training loop, and infrastructure to run it, none of which a hand-written prompt
requires. For a one-off task, or one without a labeled dataset, prompt engineering is
still the right first move — it costs nothing but iteration time.

## Where it sits on the spectrum

Prompt tuning is one point on a spectrum of techniques that trade an increasing amount
of compute and data for an increasing amount of control over model behavior:

- **Prompt engineering** — no training, discrete text, cheapest and fastest to iterate, bounded by what language alone can specify.
- **Prompt tuning** — trains a small set of continuous embeddings per task, frozen base model, needs labeled data and a training loop.
- **Prefix-tuning and p-tuning** — close relatives that train small continuous vectors injected at every transformer layer (not just the input) for more expressive control, at similar cost.
- **LoRA** — trains small low-rank update matrices inside the model's own weight matrices rather than prepending anything, still with the base weights frozen.
- **Full fine-tuning** — updates every weight in the model, the most capable and most expensive option, covered in the next lesson.

None of these are covered in depth here — the point is to place prompt tuning
correctly: well past plain prompting in terms of what it requires, well short of full
fine-tuning in terms of what it costs. If a task has a clear, stable definition and a
labeled dataset but doesn't justify retraining an entire model, prompt tuning is
usually the first PEFT technique worth trying before reaching for something heavier.
