---
title: "Writing a Reviewable Pull Request"
description: How to size, self-review, and describe a pull request so a reviewer can actually hold the change in their head.
roadmap: code-review
node: writing-a-reviewable-pull-request
group: code-review-practice
video:
  provider: youtube
  id: "nsNEUM4kk_s"
  start: 0
  end: null
creator:
  name: Google TechTalks
  channel_url: https://www.youtube.com/@GoogleTechTalks
duration_seconds: 944
level: beginner
prerequisites:
  - why-code-review-matters
takeaways:
  - "A pull request that bundles two unrelated changes isn't one review, it's two reviews wearing a trench coat — the reviewer has to mentally separate them anyway, except now without the benefit of two focused diffs and two commit messages explaining each one."
  - "Reading your own diff before requesting review, as if you were the reviewer seeing it cold, catches the debug prints and dead code a reviewer would otherwise have to flag and you'd have to fix in a second round trip."
  - "A description that just restates the diff — \"adds a null check in parseConfig\" — tells the reviewer nothing they couldn't read themselves; what they actually need is the intent the diff doesn't carry: what you were trying to fix and why this was the fix."
  - "Reviewer attention is a fixed, depleting resource per sitting — not a budget that scales linearly with how much time you give them, which is why a 1,500-line change doesn't get a review seven times as careful as a 200-line one; it gets a worse one."
  - "Running the change yourself before asking anyone to review it is the cheapest bug fix available — it costs you a few minutes and costs a reviewer nothing, versus costing a reviewer the time to notice the same problem and bounce it back."
check_questions:
  - q: Why is a pull request that mixes two unrelated changes worse than two separate pull requests, even though the total amount of code is the same?
    a: The reviewer still has to do the work of separating the two changes mentally, except now without a focused diff or a commit message for each one, and without the ability to approve the simple one while still asking questions about the riskier one.
  - q: What's wrong with a PR description that just restates what the diff already shows?
    a: The reviewer can already read the diff — what they can't read from it is why you made this change and what you were trying to accomplish, so a description that only repeats the code gives them nothing beyond what they already had.
  - q: Why does self-reviewing your own diff before requesting review matter, given that a reviewer will look at it anyway?
    a: Catching your own leftover debug print or dead code costs you a few seconds while you're already in the file, but if a reviewer catches it instead it costs their attention, your time to fix it, and a full extra round trip before the change can land.
  - q: Why does a 1,500-line pull request tend to get reviewed worse than three 200-line ones covering the same total change?
    a: A reviewer can hold a few hundred lines of context in working memory at once, but a change several times that size exceeds it, which pushes them toward skimming for an overall shape instead of checking each piece — the review gets faster and shallower exactly when the size says it should get more careful.
deeper:
  - label: "The Art of the Pull Request — Ben Lomax"
    url: "https://www.youtube.com/watch?v=2upWetU_vUc"
related_formulas: []
---

## The review starts before anyone else looks at it

It's tempting to think of code review as something that happens to your pull request once you've opened it — the reviewer's job, not yours. Rachel Fenichel's talk pushes back on that: a large share of what makes a review good or bad is decided by the author, before a reviewer ever opens the diff. How you scope the change, how carefully you check it yourself, and what you write in the description all shape whether the reviewer can actually do their job well. Get those three right and the review is often quick and useful. Get them wrong and even a careful reviewer ends up rubber-stamping something they didn't really have the capacity to check.

## One logical change, not a grab-bag

The single biggest lever is scope. A pull request should be one coherent change — not a bug fix that also happens to rename a few variables you found annoying, not a new feature bundled with an unrelated refactor you'd been meaning to do. Each of those deserves its own pull request, with its own description explaining just that one thing.

The reason isn't tidiness for its own sake. A reviewer looking at a mixed PR has to separate the changes themselves before they can evaluate either one — they can't approve the safe variable rename while still raising concerns about the risky bug fix, because the two are stuck together. If something goes wrong after merge, there's no way to revert just the feature without also reverting the fix it got bundled with. Keeping a PR to one logical change means the reviewer's job matches the one question they're actually trying to answer: is this one thing correct and well-justified.

## Review your own diff first

Before asking anyone else to look at a change, look at it yourself — not by rereading your editor, but by opening the actual diff the same way a reviewer will see it. This surfaces things you'd never notice while writing but that jump out immediately in a diff view: a leftover `print(x)` from debugging, a commented-out block you meant to delete, an import you added and never used.

None of these are hard to catch. They're hard to catch while you're writing, because you're focused on making the logic work, not on what the final diff looks like. Looking at the diff cold, as if someone else wrote it, catches them in a couple of minutes. Skipping this step doesn't make the problem disappear — it just moves the job of finding it onto the reviewer, who now has to flag it, wait for you to fix it, and look again.

## Write the intent, not a caption for the diff

A description like "updates `parseConfig` to handle missing fields" tells a reviewer only what they could already see by reading the diff. What they can't get from the diff is why: what problem you ran into, what you considered and ruled out, what you're not fully sure about. A description that says "users on older config versions were crashing on startup because `parseConfig` assumed every field was present — this adds a default path for anything missing, matching what the loader already does for the v2 format" gives the reviewer the frame to evaluate the change against, instead of just the code itself.

This is also where you say what you tested and how, and flag anything you're deliberately leaving out of scope. A reviewer who knows what you tried doesn't waste time asking about something you already ruled out.

## Why size is the thing that breaks everything else

All of this is also a defense against a problem that shows up constantly in practice: a pull request that's grown too large for anyone to review properly. A reviewer's attention for a single sitting is roughly fixed — somewhere around a few hundred lines they can actually hold in their head and check carefully. A 1,500-line change doesn't get seven times the scrutiny a 200-line change would; past a certain size, reviewers stop checking each piece and start skimming for an overall shape, because that's the only way to get through it at all. The incentive quietly flips from "check this carefully" to "this looks fine, approve it" — not because the reviewer is careless, but because the size of the ask made careful review impossible. Splitting that same work into three smaller pull requests doesn't just make each one easier to read — it keeps the review doing the job it's supposed to do.
