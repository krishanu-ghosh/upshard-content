---
title: "Common Code Review Pitfalls"
description: The recurring ways daily code review breaks down — bikeshedding, rubber-stamping, bottleneck reviewers, scope creep — and the structural fix for each.
roadmap: code-review
node: common-code-review-pitfalls
group: code-review-practice
video:
  provider: youtube
  id: "0t4_MfHgb_A"
  start: 0
  end: null
creator:
  name: PyGotham 2018
  channel_url: https://www.youtube.com/@PyGotham2018
duration_seconds: 1369
level: beginner
prerequisites:
  - automating-the-easy-checks
takeaways:
  - "Bikeshedding happens because trivial details are easy to have a confident opinion about, so reviewers unconsciously gravitate to arguing about a variable name while a real correctness risk sails through with no comment at all."
  - "Rubber-stamping is usually a symptom of a diff too large to hold in working memory, not a reviewer who doesn't care — the fix is keeping pull requests small, not asking people to concentrate harder."
  - "A single reviewer who becomes the de facto required approval for everything turns into a queue, and a queue either backs up the whole team's velocity or gets skimmed to keep up — both outcomes defeat the point of having that reviewer."
  - "Scope creep inside a review thread turns a focused correctness check into an open-ended redesign negotiated in comments, which is the wrong format for that conversation and usually means the request should become its own ticket."
check_questions:
  - q: Why do reviewers sometimes spend more comments on a variable name than on whether the algorithm around it is correct?
    a: Because a variable name is something anyone can form a confident opinion about in five seconds, while judging whether an algorithm is correct takes real effort and carries the risk of being wrong — Parkinson's law of triviality predicts exactly this, that the easy-to-argue-about detail absorbs the discussion while the substantive risk goes unexamined.
  - q: What actually causes rubber-stamping, and what fixes it?
    a: Rubber-stamping is usually caused by a diff too large to hold in working memory, not by a careless reviewer — past a certain size nobody can genuinely track every change, so approval becomes a formality; the fix is keeping pull requests small enough to actually read, not telling reviewers to try harder or read more slowly.
  - q: Why does routing every pull request through one trusted reviewer eventually hurt the team, even though that reviewer is good at the job?
    a: Because that reviewer becomes a queue that every change has to pass through — the team's shipping speed gets capped at whatever rate one person can review, and when the queue backs up the reviewer either becomes the team's bottleneck or starts skimming to keep pace, which quietly degrades the quality the setup was supposed to protect.
deeper:
  - label: "Better Code Reviews FTW! — Tess Ferrandez-Norlander"
    url: "https://www.youtube.com/watch?v=2vacqx2ISms"
related_formulas: []
---

## Knowing the right way doesn't stop these from happening

Everything covered earlier in this roadmap — what to prepare before requesting review, how to give feedback that's specific and kind, what a reviewer should actually be looking for, automating the mechanical stuff — describes code review done well. None of it makes a team immune to the ways review actually breaks down once it's running every day, on every pull request, for years. Michael Lynch's talk "How to Do Code Reviews Like a Human" is useful precisely because it catalogs the failure modes that show up *after* a team already knows the theory. Each one has a structural cause, and each one has a fix that isn't "try harder."

## Bikeshedding: the trivial detail eats the discussion

Parkinson's law of triviality describes a committee that spends an hour debating the color of the bike shed and five minutes approving the nuclear reactor plans, because anyone can have a confident opinion about paint color and almost nobody feels qualified to argue about reactor design. Code review reproduces this exactly. A reviewer can look at `tmp` versus `pending_user_count` and instantly know which one they prefer. Judging whether a caching strategy introduces a subtle staleness bug takes real effort, and being wrong about it is embarrassing. So the thread fills up with naming suggestions while the caching logic gets an approval and no comments.

The fix isn't asking reviewers to have more discipline about what they comment on — that's asking people to fight their own cognitive shortcuts through willpower, which doesn't hold up under deadline pressure. The fix is removing the trivial target before a human ever sees it: this is exactly what automating the easy checks, covered in the previous lesson, is for. If the formatter already decided the variable's casing and the linter already flagged the genuinely dead code, there's no bikeshed left to argue about.

## Rubber-stamping: approval without reading

A reviewer approving a pull request they didn't actually read looks like a discipline problem, but it's almost always a size problem. Human working memory holds a handful of things at once. A twelve-line diff touching one function is fully readable — you can hold the whole change in mind alongside what it's supposed to do. A six-hundred-line diff touching nine files is not; past some threshold, nobody is genuinely tracking all of it, and "LGTM" becomes a ritual rather than a judgment.

Telling reviewers to slow down and concentrate harder doesn't fix this, because the underlying problem — more information than working memory can hold — doesn't go away just because someone tries harder. What fixes it is the same thing that fixes most review problems: keep the diff small enough to actually read. A pull request that does one thing is one a reviewer can genuinely evaluate, which is the real argument for splitting work into small, focused changes rather than large batches.

## The single bottleneck reviewer

Every team ends up with one person who writes the clearest, most useful review comments. The natural response is to route everything through them. This works for a while and then becomes the team's ceiling: the team's shipping speed is now capped at whatever rate one human can review code, and every pull request waits in that person's queue regardless of how fast everyone else is writing code. When the queue gets long enough, that reviewer has exactly two options — become the visible bottleneck everyone complains about, or start skimming to clear the queue, which quietly turns them into a rubber stamp with a good reputation. Spreading review responsibility across more of the team, even if the average review gets slightly less sharp, usually beats concentrating it in one person who eventually has to choose between those two bad options.

## Scope creep: "while you're in there"

A review thread is the wrong place to redesign a feature. "This is good, but while you're in there, can you also refactor the module it calls" turns a focused check of one change into an open-ended negotiation with no clear end state, conducted in a comment box that wasn't built for that conversation. The author came in expecting feedback on what they wrote, not a new, unscoped assignment attached to it. The fix is a simple reflex: a suggestion that goes beyond the original change becomes its own ticket, filed and prioritized separately, not a blocking demand on the pull request already in flight.
