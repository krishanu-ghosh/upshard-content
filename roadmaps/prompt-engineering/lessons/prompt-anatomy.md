---
title: Prompt anatomy
description: The structural parts a well-built prompt is made of, and how labeling them clearly keeps a long prompt from confusing the model.
roadmap: prompt-engineering
node: prompt-anatomy
group: basic-techniques
video:
  provider: youtube
  id: "ysPbXH0LpIE"
  start: 0
  end: null
creator:
  name: Anthropic
  channel_url: https://www.youtube.com/@anthropic-ai
duration_seconds: 1492
level: beginner
prerequisites: []
takeaways:
  - A well-built prompt separates into distinct parts — role, task, context, input data, output format, and sometimes examples — rather than being one undifferentiated block of text.
  - Clearly labeling each part, with headers or tags, helps a model parse a long prompt correctly, the same way headings and paragraphs help a human reader.
  - The anatomy is model-agnostic — the same structure holds whether you label sections with XML-style tags, markdown headers, or plain labels, as long as the boundaries are unambiguous.
check_questions:
  - q: Why does labeling sections of a long prompt matter, even if a human could read it fine without labels?
    a: A model processes the prompt as one continuous sequence of tokens with no inherent sense of where an instruction ends and reference data begins, so explicit boundaries prevent it from misreading part of the context as part of the task, or vice versa.
  - q: What is the difference between the "context" and "input data" sections of a prompt?
    a: Context is background the model needs to understand the task correctly, such as domain conventions or prior decisions; input data is the specific material the task is to be performed on, such as the actual document being summarized or the actual record being classified.
deeper: []
related_formulas: []
---

## A prompt is not one block of text

Every technique covered so far in this roadmap — zero-shot instructions,
few-shot examples, chain-of-thought reasoning — assumes a prompt has
identifiable parts: an instruction, maybe some examples, maybe the thing
being acted on. As a prompt grows past a sentence or two, keeping those parts
distinct rather than running them together stops being a nicety and starts
being the difference between a prompt that works reliably and one that
doesn't. A production-grade prompt is usually built from a consistent set of
structural pieces, assembled in the same order every time.

## The six parts

**Role.** A short statement of who the model should act as, or what
perspective to bring — "You are an expert copy editor" or "Act as a senior
backend engineer reviewing a pull request." This sets tone and the kind of
judgment calls the model makes, more than it grants any real capability.

**Task.** The actual instruction: what to do, stated as directly and
specifically as the zero-shot lessons in this roadmap already covered.
Vague verbs ("help with," "look at") belong here least of all — "rewrite,"
"classify," "extract," and "summarize in three sentences" each commit to a
concrete, checkable action.

**Context.** Background the model needs in order to do the task correctly
but that is not itself the material being worked on — house style rules,
a decision made earlier in a conversation, the audience the output is for,
domain conventions a model wouldn't otherwise know.

**Input data.** The specific material the task operates on: the document to
summarize, the code to review, the record to classify. This is the one
section whose content changes on every call, which is exactly why it needs
a clear boundary — mixing it into the instruction risks the model treating
part of the data as part of the task, or the reverse.

**Output format.** An explicit specification of the shape the answer should
take, covered at length in the zero-shot and few-shot lessons: length,
structure, fields, tone. Stated separately from the task description, it is
easier to change the format without touching the instruction itself.

**Examples**, when the task calls for them. Few-shot demonstrations slot in
as their own clearly bounded section, distinct from the live input data they
are meant to illustrate a pattern for, not to be confused with.

Not every prompt needs all six — a one-line zero-shot classification prompt
doesn't need a role or a context section — but as a prompt's job gets more
specific or higher-stakes, each missing part is a specification the model has
to guess at instead.

## Why labeling the boundaries matters

A model reads a prompt as one continuous sequence of tokens with no inherent
sense of where one part ends and the next begins — it has to infer section
boundaries the same way a reader would with no paragraph breaks, except with
less tolerance for ambiguity, because a misread boundary doesn't just make the
prompt harder to follow, it can make the model act on the wrong thing
entirely. A block of reference text with no clear marker can get treated as
part of the instruction; an example meant purely as a pattern to imitate can
get treated as additional context to reason about literally.

The fix is explicit, consistent markers around each part — XML-style tags
like `<context>...</context>` and `<input>...</input>`, or plain markdown
headers like `## Task` and `## Input`, used the same way every time. Which
convention you pick matters far less than using one at all and sticking to
it: consistency is what lets a model (and a future version of yourself
editing the prompt) tell at a glance where one section stops and the next
starts, especially once a prompt is long enough that a reader can no longer
hold the whole thing in view at once.

## Order and repetition

Two smaller details compound this structure's reliability. Putting the task
instruction both near the top, to orient the whole prompt, and again right
before the output format at the end, to anchor it immediately before
generation begins, measurably reduces a model losing track of the actual ask
inside a long prompt stuffed with context and examples. And when a
single detail is critical to get right — a hard constraint, a safety
boundary, a formatting rule that absolutely cannot be violated — restating it
once more near the end, right where generation starts, costs a few tokens
and meaningfully reduces the chance it gets lost in everything that came
before it.

None of this anatomy is specific to any one model or vendor. The techniques
that follow in this roadmap — structured output, system and role prompting,
defenses against injection — are all, in effect, refinements of one or more
of these six parts, which is why having a clear mental model of the parts
themselves pays off well beyond this single lesson.
