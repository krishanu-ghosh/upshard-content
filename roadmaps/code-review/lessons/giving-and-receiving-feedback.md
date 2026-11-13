---
title: "Giving and Receiving Feedback"
description: How to phrase review comments without softening the substance, and how to read them without taking them personally.
roadmap: code-review
node: giving-and-receiving-feedback
group: code-review-practice
video:
  provider: youtube
  id: "Ea8EiIPZvh0"
  start: 0
  end: null
creator:
  name: try! Swift Conference
  channel_url: https://www.youtube.com/@trySwiftConference
duration_seconds: 1496
level: beginner
prerequisites:
  - what-reviewers-look-for
takeaways:
  - "Compassionate and candid aren't opposites that have to be traded off — kindness changes how a problem is phrased, not whether it gets raised, and a real problem stated gently is still a real problem stated."
  - "A review comment phrased as a command (\"this is wrong\") reads as a verdict with no room to respond, while the same observation phrased as a question (\"what happens if this is null?\") invites the author into a conversation about a specific case."
  - "Treating every comment as equally urgent trains authors to either fight every nitpick or ignore every blocking issue — marking which is which is part of the reviewer's job, not an optional courtesy."
  - "Silently pushing a new commit in response to a review comment leaves the reviewer unable to tell whether their point was addressed, misunderstood, or dismissed — a reply costs one line and closes that gap."
check_questions:
  - q: What does it mean for a code review to be "compassionate yet candid," and why isn't that a contradiction?
    a: It means delivering a real, substantive critique with tone that assumes good intent and respects the author, rather than either softening the critique into vagueness or delivering it bluntly without regard for how it lands — the compassion is in the delivery, the candor is in the content, and neither one requires giving up the other.
  - q: Why does phrasing a review comment as a question usually work better than phrasing it as a command?
    a: A command like "this is wrong" asserts a conclusion and leaves the author nothing to do but comply or argue, while a question like "what happens if this list is empty?" points at the same problem but invites the author to reason through it, which usually gets to the same fix with less friction.
  - q: Why should an author respond to every review comment, even ones they aren't going to act on?
    a: A reviewer who leaves a comment and then sees a new commit with no reply has no way to tell whether the point was understood, fixed, or deliberately rejected — a short reply, even just stating the reason for not changing it, keeps the conversation legible and prevents the same comment from being raised again next time.
deeper:
  - label: "David Andersson: Giving and Receiving Great Feedback through PRs"
    url: "https://www.youtube.com/watch?v=vAoeeqVdTBc"
related_formulas: []
---

## The tension the title is naming

"Compassionate" and "candid" sound like they're pulling in opposite directions — be nice, or be honest, pick one. April Wensel's talk argues they aren't actually in tension, because they operate on different parts of the comment. Candor is about the *content*: did you actually say the real problem, or did you hint at it and hope the author infers the rest? Compassion is about the *delivery*: did you say it in a way that assumes the author is a competent person who made a reasonable call given what they knew, rather than someone who needs to be corrected? A comment can be fully honest about a bug and still be delivered with respect. A comment can also be phrased so gently that the actual problem never gets stated — that's not compassion, that's a missed review.

## Phrasing comments as questions, not verdicts

"This is wrong" tells the author what you concluded. "What happens if `items` is empty here?" tells them what you noticed and lets them walk the same path you did. Both might end at the same fix, but the second one treats the author as someone capable of finding the bug themselves once it's pointed at — which is usually true, and which avoids the defensive reflex that "this is wrong" tends to trigger. The question form also protects the reviewer: maybe there's a guard clause three lines up that handles the empty case, and the reviewer missed it. A verdict delivered confidently and then retracted costs more trust than a question that turns out to have an easy answer.

This applies beyond bugs. "Why build this as a new class instead of extending the existing one?" raises a real design question without assuming the author is wrong — maybe they have a reason the reviewer doesn't know yet.

## Marking the blocking issues apart from the nitpicks

Not every comment carries the same weight, and pretending otherwise is its own failure mode. A missing null check that will throw in production is not the same category of problem as a variable name the reviewer would have picked differently. If both show up in the review with the same tone and the same lack of a label, the author has to guess which ones actually block the merge — and either argues with all of them or, worse, starts ignoring all of them. Prefixing optional comments ("nit: I'd rename this to `userCount`") costs nothing and tells the author exactly what's required versus what's a suggestion they're free to skip. It's a small habit that removes a lot of friction from the rest of the exchange.

## Explaining why, not just asserting what

"Use a constant here" is an instruction. "Use a constant here — this literal shows up in three other files, and the last time it changed, one of them got missed" is a reason. The second version does two things the first doesn't: it gives the author enough context to agree with the request instead of just complying with it, and it gives them enough information to push back if the reasoning doesn't actually apply to their case. Reviewers who only assert, never explain, end up training authors to treat review comments as arbitrary — something to satisfy, not something to reason about.

## Receiving feedback without reading it as a verdict on you

The author's half of this exchange is just as learnable as the reviewer's half. The first habit worth building: a comment on the code is not a comment on your competence, even when it points at something you clearly should have caught. Everyone ships an off-by-one error or a forgotten edge case eventually; a review existing to catch it is the system working, not a personal failure.

The second habit: reply to every comment. Pushing a new commit with no response leaves the reviewer guessing whether their point landed, got misread, or got ignored on purpose. "Done" is a complete reply. So is "not changing this because the caller already validates it upstream" — that's not insubordination, it's giving the reviewer the same chance to push back that they gave you. If a comment is genuinely unclear, ask what it means rather than guessing and hoping the next commit happens to match what they meant — guessing wrong wastes another full round trip that a single clarifying question would have skipped.

Pushing back on a comment, with a reason, is a normal part of this conversation, not a breach of etiquette. The goal on both sides isn't agreement for its own sake — it's getting to the version of the code that's actually correct, with the reasoning for every change visible to whoever reads the thread later.
