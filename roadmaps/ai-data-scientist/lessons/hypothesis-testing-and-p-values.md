---
title: Hypothesis Testing and P-Values
description: What a null hypothesis and a p-value actually mean, and the common misreadings that lead teams to the wrong conclusion.
roadmap: ai-data-scientist
node: hypothesis-testing-and-p-values
group: statistics-for-data-science
video:
  provider: youtube
  id: "vemZtEM63GY"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 681
level: intermediate
prerequisites: [probability-for-data-science]
takeaways:
  - "A p-value is the probability of seeing data this extreme (or more) if the null hypothesis were true, not the probability that the null hypothesis is true."
  - A small p-value means the observed result would be unusual under \"no effect,\" which is evidence against the null, not proof of the alternative.
  - Statistical significance is not the same as practical importance, a tiny real-world effect can still produce a small p-value with enough data.
  - Running many tests and reporting only the significant ones (p-hacking) inflates the false positive rate far above the stated threshold.
check_questions:
  - q: What does a p-value of 0.03 actually tell you?
    a: It tells you that, if the null hypothesis were really true, you'd see a result this extreme or more extreme only 3% of the time by chance alone, which is evidence against the null but not a direct measure of how likely the null or the alternative hypothesis is to be true.
  - q: "Why can a result be \"statistically significant\" but not actually matter?"
    a: With a large enough sample, even a trivially small real-world difference, like a 0.1% increase in click-through rate, can produce a very small p-value, so significance only tells you the effect probably isn't exactly zero, not that it's big enough to act on.
  - q: Why does running 20 independent tests and reporting the one significant result at p less than 0.05 mislead people?
    a: If the null hypothesis is actually true for all 20 tests, you'd still expect about one of them to come back significant by chance alone at the 5% threshold, so cherry-picking the one significant result out of many tested presents normal statistical noise as if it were a real finding.
deeper:
  - label: "Hypothesis Testing and The Null Hypothesis, Clearly Explained!!!"
    url: "https://www.youtube.com/watch?v=0oc49DyA3hU"
related_formulas: []
---

## Setting up the test: null and alternative hypotheses

Hypothesis testing starts by stating two competing claims. The **null hypothesis** (H0) is the boring, skeptical default: "there's no effect, no difference, nothing interesting going on." The **alternative hypothesis** (H1) is the claim you actually suspect: "there is a difference." Crucially, the test is designed to assess the null, not the alternative directly. You assume the null is true, see how surprising your actual data would be under that assumption, and decide whether that assumption still looks reasonable.

Worked example: your checkout page currently converts at 10%. You redesign the button and want to know if the new version is better. H0: the new button's true conversion rate is also 10% (the redesign did nothing). H1: the new button's true conversion rate is different from 10%. You then run the experiment and look at the actual data through the lens of "if H0 were true, how likely is what I just saw?"

## What a p-value actually measures

The **p-value** is the probability of observing a result at least as extreme as what you got, calculated under the assumption that the null hypothesis is true. That's a precise and narrow claim, and it's where most misunderstandings start.

Worked example, continued: say you run the new button past 1,000 visitors and get 128 conversions (12.8%), versus the old 10% baseline. A statistical test (a two-proportion z-test, mechanically) asks: if the true rate were really 10%, how often would a sample of 1,000 produce 128 or more conversions just from random sampling noise? Suppose that calculation returns p = 0.02. That means if nothing had actually changed, you'd see a result this good or better only 2% of the time by chance. That's unlikely enough that most teams would reject the null and conclude the redesign probably did something.

Notice what p = 0.02 does *not* mean. It does not mean "there's a 2% chance the null hypothesis is true" or "there's a 98% chance the new button is better." The p-value is computed by assuming the null is true in the first place, so it can't simultaneously tell you the probability that assumption is false, that would be circular. It's a statement about how surprising your data is under one specific assumption, not a probability statement about the hypotheses themselves.

## The threshold, and what crossing it does and doesn't prove

Teams typically pick a significance level (alpha) in advance, commonly 0.05, and reject the null if p falls below it. This threshold is a convention, not a law of nature, chosen to balance how often you're willing to falsely declare an effect that isn't really there (a **false positive**, or Type I error) against how often you'll miss a real effect (a **Type II error**, false negative). Lowering alpha to 0.01 makes false positives rarer but makes it harder to detect real, smaller effects with the same data.

Crossing the threshold tells you the result is unlikely to be pure noise. That's it. It does not tell you the effect is large, important, or worth shipping. With enough sample size, almost any nonzero effect becomes statistically significant eventually, because larger samples shrink the random noise around your estimate. A redesign that moves conversion from 10.00% to 10.05% can be "significant" with a million visitors despite being commercially irrelevant. The question "is this real" (significance) and the question "is this big enough to matter" (effect size) are separate. A responsible analysis reports both.

## Common misuses that produce false confidence

Two patterns deserve specific attention because they're easy to fall into without realizing it. First, **p-hacking**: running many comparisons (different metrics, different segments, different time windows) and reporting only the one that happened to cross p < 0.05. Since a true null hypothesis still produces a "significant" result about 5% of the time by pure chance, testing 20 unrelated things and highlighting the one hit is close to guaranteed to produce a false story, even with no real effect anywhere. Second, **stopping early**: peeking at results mid-experiment and stopping as soon as p dips below 0.05, rather than committing to a sample size in advance, inflates the false positive rate because you're giving random fluctuation many chances to cross the line. Both problems are solved the same way: decide your test, your metric, and your sample size before looking at the data, and treat any post-hoc significant finding you stumble on as a hypothesis to test again, not a conclusion to ship.
