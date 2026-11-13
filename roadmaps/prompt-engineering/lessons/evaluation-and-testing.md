---
title: Evaluation and testing
description: How to know a prompt actually works, and the three eval methods that trade speed and cost against the kinds of mistakes each one catches.
roadmap: prompt-engineering
node: evaluation-and-testing
group: reliability
video:
  provider: youtube
  id: "-sL7QzDFW-4"
  start: 0
  end: null
creator:
  name: Shaw Talebi
  channel_url: https://www.youtube.com/@ShawhinTalebi
duration_seconds: 1639
level: intermediate
prerequisites: [chain-of-thought-prompting]
takeaways:
  - Eyeballing a few outputs doesn't scale and doesn't catch regressions; a small, representative eval set run consistently is what tells you whether a prompt change actually helped.
  - Code-based checks, human review, and LLM-as-judge each catch different kinds of mistakes at different costs, and most real evaluation pipelines combine more than one.
  - LLM-as-judge needs its own validation against human judgment before you trust it, because judge models carry predictable biases of their own, like favoring longer or more confident-sounding answers.
check_questions:
  - q: Why doesn't trying a prompt a few times and checking if the outputs "look good" scale as a testing strategy?
    a: It only samples a handful of inputs, misses edge cases and regressions, and depends on subjective judgment that doesn't produce a comparable score across prompt versions. It can't tell you whether a new version of a prompt is actually better or just better on the few examples you happened to try.
  - q: What is the main risk of relying on LLM-as-judge without validating it?
    a: Judge models carry their own systematic biases, such as favoring longer or more confident-sounding responses regardless of correctness, so an unvalidated judge can consistently reward the wrong thing. It needs to be checked against human judgment on a sample before being trusted at scale.
deeper:
  - label: "Dave Ebbelaar — How to Systematically Setup LLM Evals (Metrics, Unit Tests, LLM-as-a-Judge)"
    url: https://www.youtube.com/watch?v=a3SMraZWNNs
related_formulas: [perplexity]
---

## "It looked good" is not a testing strategy

Every technique earlier in this roadmap — few-shot examples, chain-of-thought,
system prompting, structured output — produces a prompt that works on the
handful of inputs you tried while writing it. That tells you almost nothing
about how it performs on the inputs you didn't try, and nothing at all about
whether your next edit made things better or worse. Without a way to measure
a prompt's actual performance, "improving" a prompt is guesswork dressed up as
iteration: you change the wording, run it once or twice, decide it feels
better, and ship it — with no way to know whether you fixed the problem,
broke something else, or just got lucky on the two examples you checked.
**Evaluation** is what replaces that guesswork with a number you can compare
across versions.

## Code-based evals: fast, cheap, and narrow

The cheapest and most reliable evaluation method is a direct, programmatic
check against a known-correct answer: does the output parse as valid JSON
matching the expected schema, does it contain the exact string you asked for,
does a math answer equal the right number, does a classification label match
the ground truth. **Code-based evals** run in milliseconds, cost nothing per
run beyond the model call itself, and give an unambiguous pass or fail with no
room for interpretation. Their limitation is scope: they only work when
correctness is checkable by a program, which rules out most open-ended writing
tasks, nuanced summaries, or anything where "good" is a matter of judgment
rather than an exact match. A prompt that extracts structured data from an
invoice is a great fit. A prompt that writes a persuasive product description
is not.

## Human evaluation: the gold standard, and the bottleneck

For tasks where quality is inherently subjective — tone, persuasiveness,
helpfulness, whether a summary captures what actually mattered — a human
reading the output and scoring it against a rubric is still the most reliable
signal available. **Human evaluation** catches things no automated check will:
a technically correct answer that's condescending in tone, a summary that's
accurate but buries the one fact that mattered, a response that's well-formed
but subtly off in a way only domain expertise would catch. Its cost is the
obvious tradeoff — a person reviewing outputs doesn't scale to the hundreds or
thousands of test cases you'd want to run before every prompt change, and
getting consistent scores from multiple human reviewers requires a clear
rubric and some calibration between them, or their judgments drift apart.
Human eval is usually reserved for a smaller, carefully chosen sample, or for
validating that a cheaper automated method is actually measuring the right
thing.

## LLM-as-judge: scalable, but needs its own checking

Between those two extremes sits **LLM-as-judge**: using a second model call to
score the first model's output against a rubric you define, the way a human
rater would, but at the speed and cost of an API call rather than a person's
time. This scales to large test sets cheaply and can evaluate genuinely
subjective qualities that a code-based check can't touch. The catch is that a
judge model carries its own systematic biases — well-documented tendencies to
favor longer responses over shorter ones regardless of quality, to rate
confident-sounding answers higher even when they're wrong, and to be
influenced by the order candidate answers are presented in when comparing two
outputs head to head. None of this means LLM-as-judge is untrustworthy; it
means it needs to be validated the same way any measurement tool does — run it
against a sample where you also have human scores, check that the two agree
often enough to trust the judge on the rest, and design the judge's rubric
specifically to resist the known biases (for instance, explicitly telling it
not to favor length).

## Build a small eval set and let it drive changes, not just confirm them

The practical shape of all this, for a prompt you actually care about, is a
small, fixed set of representative test cases — real or realistic inputs
covering the common case and the edge cases you know matter — scored by
whichever combination of these three methods fits the task. Run every
candidate prompt against the same set, compare the scores, and only ship a
change that measurably improves them. The discipline that matters most isn't
which method you pick; it's treating the eval set as the thing that decides
whether a prompt change is good, rather than something you consult after
you've already decided to ship it.
