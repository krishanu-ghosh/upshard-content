---
title: User Testing and Feedback Loops
description: Running real user conversations that surface honest signal instead of politeness, and feeding what you learn back into the next build cycle.
roadmap: ai-product-builder
node: user-testing-and-feedback-loops
group: testing-and-quality
video:
  provider: youtube
  id: "z1iF1c8w5Lg"
  start: 0
  end: null
creator:
  name: Y Combinator
  channel_url: https://www.youtube.com/@ycombinator
duration_seconds: 1051
level: beginner
prerequisites:
  - end-to-end-testing
takeaways:
  - A passing automated test suite only confirms the app works mechanically — it says nothing about whether a real person wants to use it, which is a separate question only real users can answer.
  - People are reflexively polite about things you built; asking "do you like it" or "would you use this" invites a kind answer, not an honest one, so the questions have to be designed around that bias rather than hoping to overcome it.
  - Asking about specific past behavior — what someone actually did the last time they faced this problem — surfaces real signal; asking about hypothetical future behavior mostly surfaces good intentions.
  - A good user conversation watches for what someone actually struggles with while using the app in front of you, not just what they say about it afterward.
  - Feedback is only useful once it changes the next build cycle — the point of this lesson is to close the loop back into the prompting, prototyping, and coding stages earlier in this roadmap, not to collect opinions as an end in themselves.
check_questions:
  - q: Why doesn't a passing end-to-end test suite tell you whether the app is actually good?
    a: It confirms the critical paths work mechanically from the outside in — that a user who tries to sign up and check out can succeed — but it says nothing about whether real people want the product, find it worth the friction, or come back to it, which only a real user can tell you.
  - q: Why does asking "would you use this?" tend to produce misleading feedback?
    a: It's a hypothetical about future behavior, and people tend to answer hypotheticals optimistically and politely rather than honestly, especially face to face with the person who built the thing; asking about a specific instance of past behavior is harder to answer with a kind guess.
  - q: What makes user feedback actually useful, beyond just collecting it?
    a: It has to feed back into the next iteration — the prompts, the scope, the parts of the app you build or rebuild next — so the conversation only pays off once it changes a concrete decision in the next build cycle, not when it's merely recorded.
deeper: []
related_formulas: []
---

## The test that automation can't run

Every lesson in this group so far has been about verifying that the app works: that functions return correct values, that the frontend and API agree, that a signup flow completes in a real browser. All of that answers one question — does the app do what it's supposed to, mechanically. It does not answer the question that actually determines whether the product is worth building: does a real person want this, understand it, and get value out of it. No test suite, however green, can answer that. Only a real user can, which is why this is the last stop in the testing group rather than the first — it's the test you run once the earlier ones stop being the bottleneck.

## People are polite, and that ruins your data

The biggest obstacle to a useful user conversation isn't getting someone to talk to you — it's that once they're talking to you, they tend to be nice about it. Ask "do you like it?" to someone looking at something you clearly built with care, and you'll mostly get encouragement, because the honest critical answer is socially uncomfortable to give a person standing right there. This isn't a flaw in any particular user; it's close to universal, and no amount of reassuring them you "really want honest feedback" fixes it, because the discomfort of criticizing someone to their face doesn't go away just because they asked for it.

## Ask about the past, not the future

The practical fix is to change what you ask rather than hoping people will answer differently. A hypothetical question about the future — "would you use this," "do you think this would help" — invites a hypothetical, optimistic answer, because agreeing costs nothing and committing to nothing is easy. A specific question about the past — "tell me about the last time you ran into this problem, what did you actually do" — is much harder to answer politely instead of honestly, because it's asking for a fact, not an opinion. The difference in signal quality between those two kinds of questions is enormous, and it's close to the single most useful habit in this lesson.

## Watch more than you ask

The other source of real signal is simply watching someone use the thing you built, rather than only asking them about it afterward. Where do they hesitate. What do they click that isn't actually clickable. What do they try to do that the app doesn't support, because they assumed it would work the way something else they use works. Those moments are data whether or not the person says anything about them out loud, and they often contradict whatever polite summary comes at the end of the session.

## Closing the loop back into the build

None of this matters unless it changes something. A user conversation that produces a page of notes nobody acts on is no better than skipping it. The point of running this at the end of a build cycle is to feed what you learned straight back into the earlier stages of this roadmap — back into scoping what the app should even do, back into how you prompt the next round of generation, back into which feature actually gets built next. The app already passed its own tests by the time you get here; the only test left that actually settles anything is a real person trying to use it, and the only way that test pays off is by changing what you build next.
