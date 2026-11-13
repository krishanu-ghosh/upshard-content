---
title: "DRY, KISS, and YAGNI"
description: Three heuristics against duplicated logic, needless complexity, and speculative work — what each one actually guards against, and where each gets misapplied.
roadmap: software-design-architecture
node: dry-kiss-and-yagni
group: design-principles
video:
  provider: youtube
  id: "eNnFThA60yw"
  start: 0
  end: null
creator:
  name: Yash Jain
  channel_url: https://www.youtube.com/@YashJainIO
duration_seconds: 1158
level: beginner
prerequisites: []
takeaways:
  - "DRY targets duplicated knowledge, not duplicated text — two functions that happen to contain the same five lines today aren't a DRY violation unless they represent the same business rule that should always change together."
  - "Merging code that merely looks similar, but actually changes for unrelated reasons, creates a false shared abstraction that breaks one caller every time you fix a bug for another."
  - "KISS means picking the simplest design that still does the job correctly, not the design with the fewest lines — skipping error handling or input validation to look simple is under-engineering wearing KISS as an excuse."
  - "YAGNI means not building for a need you're guessing at, not refusing to build for a need you already know is coming next sprint — a confirmed near-term requirement isn't speculative, it's just not written yet."
  - "DRY and YAGNI actively pull against each other: factoring out a shared abstraction to avoid future duplication is itself building for a need you don't have yet, which is exactly what YAGNI warns against."
  - "None of the three is a rule a linter can check — they're judgment calls about which cost (duplication, complexity, or upfront work) is cheaper to pay later."
check_questions:
  - q: Why doesn't identical-looking code in two places automatically violate DRY?
    a: DRY is about one piece of knowledge living in one place, not about character-for-character duplication — if the two pieces of code represent different business rules that happen to coincide today, merging them creates a false shared abstraction that will have to be split apart again the first time one of those rules changes and the other doesn't.
  - q: How does KISS get used to justify the opposite of what it means?
    a: KISS asks for the simplest design that still works, but it gets stretched to mean the design with the least code written, which excuses skipping error handling, edge cases, or input validation — that isn't simple, it's incomplete, and the missing work shows up later as a bug instead of as a line of code now.
  - q: What's an example of YAGNI being misapplied to dodge work that's actually needed soon?
    a: A team that already knows next month's release requires exporting data to CSV, but skips building any export path now because "we don't need it yet," isn't practicing YAGNI — YAGNI is for genuinely speculative features, not for a requirement that's already been confirmed and just hasn't been scheduled this sprint.
deeper:
  - label: "Do you really need that abstraction or generic code? (YAGNI)"
    url: "https://www.youtube.com/watch?v=_Al7qI4vMt0"
related_formulas: []
---

## Three different ways of saying "don't"

DRY, KISS, and YAGNI aren't one idea repeated three times — they're three separate warnings against three separate ways a codebase gets worse. DRY (Don't Repeat Yourself) guards against the same fact living in two places, so a change to one gets forgotten in the other. KISS (Keep It Simple, Stupid) guards against solving an easy problem with a complicated mechanism. YAGNI (You Aren't Gonna Need It) guards against building a feature for a need that hasn't actually shown up yet. Each one is a reaction to a specific, common way of wasting effort or creating risk — and each one gets misquoted, by beginners and veterans alike, to justify the opposite of what it was meant to prevent.

## DRY: one fact, one place — not one shape, one place

DRY is about duplicated *knowledge*, not duplicated *text*. If a `validateEmailFormat` check and a `validatePhoneFormat` check happen to both be five lines of regex-and-throw, that's not a DRY violation — they're two different facts about the business that happen to look similar today. Merging them into one `validateField(pattern, value)` function creates a dependency that didn't exist before: the day email validation needs to also check for disposable-email domains, you either bolt that logic onto a function that phone numbers also call, or you duplicate it back out anyway. The tell for a real DRY violation is that the two pieces of code would always need the same fix for the same reason — not that they currently look alike.

## KISS: the simplest thing that still works

KISS says to pick the simplest design that correctly solves the problem — the emphasis is on *correctly*. It gets misused as cover for cutting corners: skipping a null check, ignoring what happens when a network call times out, hardcoding a value that should be configurable, and calling the result "simple" because it's short. That's not simple, it's missing work. A genuinely KISS solution to "read a config file" is a plain function that reads a file and handles the file not existing — not a four-layer configuration-provider abstraction with pluggable backends nobody asked for, but also not a version that crashes the first time the file is missing.

## YAGNI: don't build for a guess, do build for a fact

YAGNI says not to build functionality speculatively, for a need you're only imagining might show up. It does not say to ignore a requirement you already know is coming. If product has confirmed that CSV export ships next month, skipping any thought toward it now because "we don't need it yet" isn't YAGNI — that's deferring confirmed work and calling it discipline. YAGNI applies to the second system you *might* need to support someday, the configuration option nobody has asked for, the abstraction layer built "in case we swap databases" when nobody has proposed swapping databases. The test is whether the need is confirmed or guessed, not whether it's needed this week or next quarter.

## Where the three pull against each other

These heuristics aren't always aligned. Removing a duplicated block (DRY) often means introducing a shared function, a parameter, or an interface — which is more machinery than either of the two original call sites had, pulling against KISS's preference for the plainest thing that works. Worse, building that shared abstraction *before* a second real need for it exists is building for a guess, which is exactly what YAGNI warns against — the common advice to "wait for the third occurrence before you abstract" (the Rule of Three) exists specifically to resolve this tension, by giving DRY a reason to wait until the duplication is proven real rather than assumed.

## Applying them as a set, not a checklist

None of these three has a mechanical trigger. A linter can't tell you whether two similar-looking functions represent the same business rule, whether a bit of extra code is covering a real edge case or just padding, or whether a requirement is confirmed or imagined. What they share is the same underlying question: is this extra structure — the shared function, the configuration knob, the abstraction layer — paying for itself right now, or is it a bet on a future that may never arrive? Treating all three as laws to satisfy on every file, rather than questions to ask about the one in front of you, is how a codebase ends up either a pile of copy-pasted logic or a maze of unused flexibility — the two outcomes these principles exist to prevent in the first place.
