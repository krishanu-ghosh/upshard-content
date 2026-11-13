---
title: Why evals matter
description: Why iterating on vibes stops working, what an eval actually is, and the error-analysis loop that turns failures into measurements.
roadmap: ai-engineer
node: why-evals-matter
group: evaluation
video:
  provider: youtube
  id: "e2i6JbU2R-s"
  start: 0
  end: null
creator:
  name: Hamel Husain
  channel_url: https://www.youtube.com/@hamelhusain7140
duration_seconds: 728
level: intermediate
prerequisites: [rag-pipeline]
takeaways:
  - Without measurement you cannot tell improvement from change, and a prompt edit that fixes the case in front of you may break four you are not looking at.
  - An eval is a dataset of inputs, a way of judging the output, and a score you can compare across versions — nothing more exotic than a test suite with fuzzy assertions.
  - Error analysis comes first — read real failures, name the categories, and let the categories tell you which evals are worth writing.
check_questions:
  - q: Why is looking at a few outputs and deciding they seem better not enough?
    a: It has no baseline and no coverage. You see the cases you chose to look at, you cannot detect regressions elsewhere, and nothing is comparable between two versions or two people.
  - q: Why does error analysis come before writing evals?
    a: Because evals written in advance measure what you imagined would break. Reading real failures tells you what actually breaks, and the failure categories you find are what the evals should target.
deeper:
  - label: "Manu Goyal — Why should anyone care about evals?"
    url: https://www.youtube.com/watch?v=jJ45Yz1lJao
  - label: "Hamel Husain — LLM evals, common mistakes"
    url: https://www.youtube.com/watch?v=GL0XhAj5LPE
related_formulas: []
---

## The point where vibes stop working

Early on, you change a prompt, try three inputs, and you can clearly see it
is better. This works, and it keeps working right up until the system has
accumulated enough behaviour that no single person can hold it all in their
head at once. Then a change that visibly fixes the case you were staring at
quietly breaks two others you were not looking at, and nobody notices for a
fortnight. The transition is not subtle in hindsight: it looks like a team
that cannot say with confidence whether last month's version was actually
better than this month's, that argues about whether a model upgrade helped
based on anecdote rather than measurement, and that becomes afraid to touch a
prompt because it has become load-bearing in ways nobody fully mapped out.

## What an eval actually is, with no mystique

Three parts, and nothing more exotic than that. A set of inputs, ideally
drawn from real usage rather than invented. A way of deciding whether a given
output is acceptable — this can be an exact string match, a deterministic
rule, a human-assigned label, or a model grading against a stated rubric,
covered in [LLM as judge](/roadmaps/ai-engineer/llm-as-judge). And a score
that means the same thing today as it meant last week, so that two different
versions of the system can genuinely be compared against the same yardstick.
That is the whole of it: a test suite whose assertions are fuzzy rather than
exact, and the fuzziness is the one genuinely new engineering problem this
field introduces on top of ordinary software testing.

## Borrowing, and departing from, the testing pyramid

Traditional software testing has unit tests at the base — fast, narrow,
deterministic — with slower, broader integration and end-to-end tests above
them. LLM evaluation borrows this shape loosely: fast deterministic
assertions, covered in
[building an eval set](/roadmaps/ai-engineer/building-an-eval-set), sit at
the base and should be preferred wherever a check can be made exact. Above
them sit evals requiring judgement — human-labelled or model-graded — which
are slower and more expensive to run, and above that sits online evaluation:
watching real production behaviour, such as whether users retry a request,
abandon a conversation, or give explicit negative feedback, as a continuous
signal rather than a one-off offline test. The genuine departure from
traditional testing is that even the base layer here is rarely binary
pass-or-fail against an exact expected output; it is closer to a set of
properties an acceptable answer must satisfy, which still has to be checked
mechanically wherever that is possible.

## Error analysis, worked through on a small example

Reading real failures before writing a single formal eval is the activity
that pays off the most in this entire field. It's also the least glamorous,
which is exactly why it tends to get deferred.

Here's a concrete illustration. Take twenty failed customer-support responses
pulled from recent traces and read them one by one. Eight involve the model
answering a question about a feature the product doesn't actually have. Six
involve the model citing an outdated pricing figure. Three involve an overly
long, rambling response where a short one was needed. Three are genuinely
miscellaneous, with no clear shared cause. That one hour of reading has just
handed you three concrete, measurable eval categories worth building —
hallucinated features, stale pricing, response length — ranked by how often
they actually occur. An eval framework installed without first doing this
reading cannot give you that prioritisation.

## Leading indicators versus lagging ones

A well-built eval set, run before shipping a change, is a leading indicator —
it predicts quality before real users are exposed to a regression. Production
metrics like user complaints, support escalations, or a drop in task
completion rate are lagging indicators — they confirm a problem only after
real users have already experienced it. Both matter, but the entire point of
investing in an eval set is to catch as much as possible with the leading
indicator, specifically so that the lagging one stays boring rather than
being the first and only signal a team gets that something broke.
