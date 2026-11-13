---
title: "Automating the Easy Checks"
description: Why formatting and style debates are the wrong place for a reviewer's attention, and how linters, formatters, and CI gates take that work off the table.
roadmap: code-review
node: automating-the-easy-checks
group: code-review-practice
video:
  provider: youtube
  id: "9psDYv4kVvE"
  start: 0
  end: null
creator:
  name: EuroPython Conference
  channel_url: https://www.youtube.com/@EuroPythonConference
duration_seconds: 1737
level: beginner
prerequisites:
  - giving-and-receiving-feedback
takeaways:
  - "A reviewer's attention is the scarce resource in code review, not the reviewer's time slot on the calendar — a 20-minute review spent on tabs-vs-spaces has 20 fewer minutes left for the one bug a human was actually needed to catch."
  - "A formatter and a linter solve two different problems — a formatter has no judgment calls to make and should just run, while a linter encodes judgment calls a team has to agree on, configure, and occasionally override."
  - "A written style guide decays silently because nobody re-reads documentation after onboarding, while a CI check that blocks a merge enforces itself every single time, with no reliance on anyone remembering it exists."
  - "Automation only catches what's mechanically checkable — syntax, formatting, known bug patterns — and is structurally incapable of judging whether the code solves the right problem, which stays entirely a human reviewer's job."
check_questions:
  - q: Why does arguing about import order in a pull request cost more than the two minutes the argument takes?
    a: Because the reviewer only has a fixed amount of attention for that diff, and every minute spent on a mechanical detail is a minute not spent checking the logic, the edge cases, or the security implications that a machine cannot evaluate — the real cost is what didn't get reviewed, not the time the comment thread took.
  - q: What is the practical difference between a formatter and a linter?
    a: A formatter like Black, Prettier, or gofmt makes no judgment calls — it has one correct output for a given input and should simply be run, never debated — while a linter like Pylint, ESLint, or golangci-lint flags patterns that genuinely require a human decision, which is why linters need real configuration and the occasional deliberate override.
  - q: Why does a style guide that everyone agreed to in a wiki page stop working within a few months?
    a: Because nobody re-reads a wiki page once they've stopped being new, so the rule only survives in the memory of whoever wrote it, and it silently stops being followed the first time a reviewer forgets it or a new hire never saw it — a CI check has no memory to lose, since it reads the same rule off the same file on every single pull request.
deeper:
  - label: "Python Linters at Scale — Jimmy Lai"
    url: "https://www.youtube.com/watch?v=-ihC3frDxss"
related_formulas: []
---

## The bottleneck is attention, not time

Think about what a code reviewer actually has a limited supply of. It isn't minutes on a calendar — most teams can find twenty more minutes somewhere. It's attention: the ability to hold the author's intent, the surrounding system, and the diff in your head at once, and notice when something is subtly wrong. That capacity drains fast, and it doesn't come back partway through a review session just because the thing you're looking at got easier.

This is the problem Luke Lee's talk "Lint All the Things!" is really about, even though it's framed around linters. Every comment a reviewer leaves about a missing semicolon, an unsorted import block, or a variable named `data` instead of something more specific is a comment that spent attention on something a computer could have flagged before the human ever opened the diff. The bug that actually mattered — the off-by-one in a pagination loop, the race condition in a cache invalidation path — gets reviewed with whatever attention is left over, which by that point is often not much.

## Two different machines for two different jobs

It helps to split "automated checks" into two categories that behave completely differently, because treating them the same is where teams go wrong.

A **formatter** — Black for Python, Prettier for JavaScript, gofmt for Go — makes no judgment calls. Given the same input, it always produces the same output. There is nothing to discuss, because there is nothing to decide; the tool already decided. The only sane policy for a formatter is to run it automatically, on save or in CI, and treat its output as non-negotiable. A pull request comment that says "please run the formatter" should never exist — a CI job should have already blocked the merge before a human saw it.

A **linter or static analyzer** — Pylint, ESLint, golangci-lint, and similar tools — is different. It flags things that need a human decision: is this unused variable dead code or a placeholder for work in progress, is this broad `except:` clause intentional, does this function really need to be this long. These tools need actual configuration, because the right answer depends on the codebase, and they need an escape hatch — an inline suppression comment with a reason attached — for the cases where the rule is wrong for this one spot. Treating a linter's suggestions as automatically correct is its own mistake; treating them as something to re-litigate from scratch in every review is the mistake this lesson is about.

## Why the wiki page doesn't survive contact with reality

Most teams' first attempt at consistency is a style guide document. It works for exactly as long as everyone who wrote it is still paying attention to it, which in practice means a few weeks. Nobody goes back and rereads a wiki page once they've internalized "most" of it. New hires skim it once during onboarding and then never again. The document itself never tells you it's gone stale — it just sits there, technically still true, increasingly ignored.

A CI check that blocks a merge has no equivalent decay path. It doesn't rely on anyone's memory. It runs the same way on pull request number four and on pull request number four thousand. The rule doesn't have to be remembered because it's enforced structurally, by the system the team already has to pass through to ship anything. This is the actual argument for automating the easy checks: not that it's more convenient, but that it's the only version of the rule that keeps working after everyone stops thinking about it.

## What this doesn't solve

None of this makes human review less necessary — it narrows what human review is for. A linter can tell you that a function is 300 lines long. It cannot tell you that the function solves a problem nobody has, or that the whole approach is wrong for the system it's going into, or that the error message it returns will confuse the person debugging it at 2 a.m. Those judgments — the ones this roadmap's lesson on what reviewers look for is actually about — are squarely human work, and they're exactly the work that automating the mechanical stuff is supposed to protect.
