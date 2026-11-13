---
title: Communicating Results and Data Storytelling
description: Structuring a narrative around an analysis so stakeholders act on it correctly, without overclaiming what the data shows.
roadmap: ai-data-scientist
node: communicating-results-and-data-storytelling
group: shipping-insights
video:
  provider: youtube
  id: "jizZKNnx9wA"
  start: 0
  end: null
creator:
  name: Jeff Su
  channel_url: https://www.youtube.com/@JeffSu
duration_seconds: 452
level: beginner
prerequisites: [llms-for-data-scientists, model-interpretability-and-explainability]
takeaways:
  - "A finding without a \"so what\" doesn't get acted on, no matter how rigorous the analysis behind it; stakeholders need to know what decision the number should change."
  - Leading with the conclusion and backing it with evidence, rather than building up to the conclusion chronologically, respects a stakeholder's time and attention.
  - "Overclaiming — stating a correlation as a cause, or a sample result as a certainty — is the fastest way to lose credibility once someone checks the number against reality."
  - Choosing what to leave out of a presentation is as much a part of data storytelling as choosing what to include, because excess detail buries the point you need understood.
check_questions:
  - q: Why does a technically correct finding sometimes fail to get acted on?
    a: Because it doesn't answer the "so what" question, it states a fact without connecting it to a decision the audience can actually make, so the audience nods and moves on without changing anything.
  - q: What's the risk of presenting an analysis's findings in the order you discovered them?
    a: It forces the audience to follow your exploratory path and wait for the payoff at the end, when most stakeholders need the conclusion first so they can decide how much of the supporting detail they actually need to see.
  - q: What's an example of overclaiming in a data presentation, and why is it risky?
    a: "Saying \"the new feature caused the retention increase\" when you only ran a correlational analysis is overclaiming, and it's risky because if anyone finds a confound, such as a concurrent marketing campaign, your broader credibility, not just this one claim, takes the hit."
deeper:
  - label: The ONLY Data Storytelling Framework You Need
    url: https://www.youtube.com/watch?v=ry4wu8ZRvUM
  - label: Why Data Storytelling is the #1 Skill Employers Want in 2025
    url: https://www.youtube.com/watch?v=YJn7Zicrg_0
related_formulas: []
---

## The analysis isn't finished until someone can act on it

A correct analysis that doesn't change a decision has failed at its actual job. This isn't a cosmetic complaint about slide design; it's a structural one. Most analyses fail to get acted on not because the math was wrong, but because the "so what" was never stated out loud. If you present a chart showing that churn is 4% higher for customers on the monthly plan than the annual plan, and stop there, you've handed the audience a fact and a homework assignment: figure out what to do with it. Most won't. The analysis needs to end with a sentence that names the implication directly: "customers who aren't offered the annual plan at signup churn at a meaningfully higher rate, which suggests we should test defaulting new signups into an annual-first flow." That sentence is the actual deliverable. The chart is the evidence for it.

## Lead with the answer, not the journey

A natural instinct, especially early in a career, is to present an analysis the way you did it: here's the data, here's what I tried first, here's what I noticed, here's a wrinkle I had to work around, and finally, here's what it means. This chronological structure is the right order for your own notebook and the wrong order for a presentation. Stakeholders don't need your exploratory path; they need the conclusion first, so they can decide how deep to go into the evidence.

A better structure inverts it: state the conclusion and the recommended action in the first sentence, then provide the two or three pieces of evidence that support it, then address the most likely objection or caveat, and only go into methodology if someone asks. This is sometimes called "answer first" structure, and it matters because attention is front-loaded in any meeting or document — the first thirty seconds get read carefully, and everything after gets skimmed. Put your most important claim where the attention actually is.

## Overclaiming: the credibility tax

The single fastest way to damage your credibility as a data scientist is to state a finding more strongly than your method supports, and get caught. The two most common forms:

**Correlation stated as causation.** "Customers who use the mobile app churn less" is a correlational finding from observed data. "The mobile app reduces churn" is a causal claim that requires something closer to a controlled experiment, or at minimum a serious attempt to rule out confounds (maybe more engaged customers are both more likely to install the app and more likely to stay, and the app isn't doing anything). If you present the causal version and someone later points out the confound, the damage isn't limited to that one slide — people start double-checking everything else you've said.

**A sample result stated as a certainty.** If an A/B test shows a 2% lift with a confidence interval that spans -1% to 5%, saying "the new design increased conversion by 2%" omits the fact that the true effect could plausibly be zero or even negative. The honest version — "the new design likely improved conversion, though the test wasn't large enough to rule out no effect" — is less punchy and more defensible. Stakeholders can handle appropriate uncertainty; what erodes trust is confident language that turns out to not have been warranted.

The discipline here is simple to state and hard to practice under pressure to sound decisive: say exactly what your method supports, no more. If you only have correlational data, say "associated with," not "caused." If your sample size is small, say so in the same breath as the headline number, not buried in a footnote.

## What to leave out

Data storytelling is as much a subtraction exercise as an addition one. A chart that overlays eight metrics at once to prove you examined everything thoroughly actually obscures the one or two trends that matter, because a reader's attention has to split eight ways. The discipline is to pick the smallest set of evidence that fully supports your conclusion, and move everything else to an appendix. If someone asks "did you check whether this varies by region," you want to be able to say "yes, here's that slide" — but you don't want that slide in the main flow if region wasn't the finding.

This same principle applies to caveats. Including every possible limitation of an analysis to appear rigorous backfires; it reads as hedging and makes the audience unsure what to trust. Pick the one or two caveats that would actually change someone's decision if they turned out to be true (a confound that could explain away the effect, a sample that might not generalize), state those clearly, and leave the rest for a follow-up conversation if someone asks. A presentation that tries to be bulletproof against every objection usually ends up persuading no one.
