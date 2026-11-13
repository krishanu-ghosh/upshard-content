---
title: Reviewing AI-Generated Code
description: Review practices for AI-written diffs, from hallucinated APIs to security issues an agent has no reason to avoid unless told to.
roadmap: ai-product-builder
node: reviewing-ai-generated-code
group: building-with-ai-coding-agents
video:
  provider: youtube
  id: "As2xy_cSx00"
  start: 0
  end: null
creator:
  name: Owain Lewis
  channel_url: https://www.youtube.com/@owainlewis
duration_seconds: 860
level: intermediate
prerequisites:
  - refactoring-and-new-features-with-ai
takeaways:
  - "\"It looks plausible\" is not the same bar as \"it's correct\" — fluent, well-formatted code is exactly what a language model produces whether or not the logic inside it is right."
  - Agents occasionally hallucinate APIs, functions, or package names that don't exist and are confidently wrong in a way a human mistake rarely looks like — this needs an explicit check, not a skim.
  - An agent has no instinct for security unless the codebase or the prompt supplies one; it will happily write a SQL query via string concatenation or skip input validation if nothing told it not to.
  - Reviewing AI-written code benefits from the same checklist every time — does it compile and run, do the tests actually cover the change, do the imports and APIs it calls actually exist, is there an obvious security gap — rather than relying on a gut read.
  - The reviewer's job doesn't shrink because an agent wrote the first draft; if anything it shifts toward verification, since the usual signal of "this took effort to write, so it was probably thought through" no longer applies.
check_questions:
  - q: Why is "the code looks plausible" a weak signal that it's actually correct?
    a: A language model produces fluent, well-formatted, idiomatic-looking code as a matter of course, regardless of whether the underlying logic is right — plausibility is about surface fluency, not about correctness, so it tells you little about whether the code actually does what it claims.
  - q: What does a hallucinated API look like, and why is it dangerous specifically in generated code?
    a: It's a function, method, or package that doesn't exist but is referenced with complete confidence and correct-looking syntax, often because it resembles something real; it's dangerous because it can pass a casual read, sometimes even survive until a dependency is installed, and only fails at the point it's actually invoked.
  - q: Why might an AI agent write a security vulnerability into otherwise working code?
    a: An agent optimizes for satisfying the request it was given, and nothing about "make this work" inherently includes "and resist an adversary" — unless the prompt or the codebase's existing patterns make secure practice the path of least resistance, the agent has no built-in reason to avoid an insecure but functional shortcut.
deeper:
  - label: "How to Catch AI Code Mistakes Before They Reach Production"
    url: "https://www.youtube.com/watch?v=BFWkVTLWBlM"
related_formulas: []
---

## Plausible is the default output, not a good sign

Language models are trained to produce text that reads well, and code is no exception — proper indentation, sensible variable names, comments that sound like they belong there. That fluency is the default texture of everything these models write, whether the logic underneath is airtight or subtly broken. A human who writes sloppy, inconsistent code is often also thinking sloppily, so scruffy formatting is a real (if imperfect) signal of carelessness worth double-checking. That correlation breaks completely with generated code: a beautifully formatted function can be wrong in a way that has nothing to do with how it looks. Reviewing AI output means consciously discounting the fluency and checking the logic directly, because the usual shortcut — "this looks careful, so it probably is" — doesn't transfer.

## Hallucinated APIs are a distinct failure mode

Occasionally an agent will call a method that doesn't exist on a class that does, import a package that was never published, or reference a configuration option that sounds right but isn't real. These aren't typos — the name is often a very plausible one, resembling a real API closely enough to pass a quick skim, especially if you're not deeply familiar with the exact library version in use. The practical defense is specific: when an agent's diff introduces a call to something unfamiliar, check that it actually exists in the installed version of the dependency, rather than trusting that correct-looking syntax implies a correct-existing target. This is a five-second check that catches a failure mode code review habits built around human-written code were never designed to look for.

## Security is nobody's default unless it's somebody's explicit job

An agent asked to "save this form data to the database" will produce code that does exactly that, as directly as possible — which can mean a SQL query built by string concatenation, user input trusted without validation, or a secret logged in plaintext, none of which the agent has any inherent reason to avoid. It isn't that the agent doesn't know better in the abstract; it's that nothing in a bare functional request invokes that knowledge. The fix is the same one from the refactoring lesson: constraints do the work instructions don't. A codebase with consistent, visible secure patterns tends to get those patterns followed; a prompt that explicitly asks for parameterized queries and input validation gets them far more reliably than one that just asks for the feature to work.

## A checklist beats a gut read

Relying on "this seems fine" scales badly, because plausibility is exactly the dimension an agent is optimized to deliver on. A short, repeatable checklist holds up better: does it actually compile and run, not just read correctly; do the tests that come with the change genuinely exercise the new behavior, or just call it once with a happy-path input; does every import and API reference resolve to something real; is there an input path that reaches a database query, a shell command, or rendered HTML without being validated or escaped. None of these checks require distrust of the tool in general — they're the same checks a careful reviewer would want run against human-written code too, just more consistently necessary here because the usual tell (code that looks like it took effort) isn't a tell anymore.

## The reviewer's job gets more important, not less

It's tempting to treat an agent's output as pre-reviewed because it came from a capable model — but the model's job was to produce something that satisfies the request, not to guarantee it's correct, secure, and complete. That verification step doesn't disappear when an agent writes the first draft; it moves entirely onto the reviewer, since there's no longer a human's own effort and care baked into the code as a first pass. Treating every AI-generated diff with the same rigor as a stranger's unreviewed pull request, rather than a trusted colleague's, is the mindset this whole lesson is really about.
