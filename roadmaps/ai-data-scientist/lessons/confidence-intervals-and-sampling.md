---
title: Confidence Intervals and Sampling
description: How sampling distributions and standard error let you build a confidence interval, and what that interval actually promises.
roadmap: ai-data-scientist
node: confidence-intervals-and-sampling
group: statistics-for-data-science
video:
  provider: youtube
  id: "TqOeMYtOc1w"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 401
level: intermediate
prerequisites: [hypothesis-testing-and-p-values]
takeaways:
  - "A sampling distribution describes how a statistic, like a sample mean, would vary if you repeated the sampling process over and over."
  - Standard error shrinks as sample size grows, specifically by a factor of the square root of n, which is why quadrupling your sample only halves your margin of error.
  - "A 95% confidence interval means the method produces an interval containing the true value 95% of the time across repeated sampling, not that there's a 95% chance the true value is in this specific interval."
  - Wider confidence intervals usually mean smaller samples or noisier data, not a more "careful" estimate.
check_questions:
  - q: What is a sampling distribution, concretely?
    a: It's the distribution you'd get if you repeated the same sampling process many times, each time computing the same statistic (like a sample mean) and recording it, which describes how much that statistic naturally bounces around from one sample to the next.
  - q: Why does doubling your sample size not cut your margin of error in half?
    a: Standard error is proportional to 1 divided by the square root of sample size, so to halve the margin of error you need to quadruple the sample size, not just double it, which is why the biggest precision gains come from small sample sizes and diminish from there.
  - q: What's wrong with saying \"there's a 95% chance the true mean is between 48 and 52\"?
    a: The true mean is a fixed, unknown number, not a random variable, so once a specific interval like 48 to 52 is calculated it either contains the true value or it doesn't; the 95% refers to how often the method of constructing such intervals succeeds across repeated samples, not the probability for this one interval.
deeper:
  - label: "The standard error, Clearly Explained!!!"
    url: "https://www.youtube.com/watch?v=XNgt7F6FqDU"
related_formulas: []
---

## Why a single sample mean is never exactly right

Every time you sample 500 users and compute their average session length, you get an estimate, not the truth. A different 500 users would give a slightly different number. This variability isn't a flaw in your method, it's an inherent feature of sampling, and understanding it is what lets you say how much to trust any single estimate.

Imagine the true (unknowable in practice) average session length across your entire user base is exactly 8.0 minutes. If you repeatedly drew samples of 500 users and computed the mean each time, you wouldn't get 8.0 minutes every time, you'd get a spread of values: 7.8, 8.3, 7.9, 8.1, and so on. The distribution of all those possible sample means is called the **sampling distribution of the mean**. A remarkable fact, the Central Limit Theorem, guarantees that this sampling distribution is approximately normal (bell-shaped) even if the underlying session-length data itself is skewed, as long as your sample size is reasonably large. That's what makes the rest of this machinery work regardless of your raw data's shape.

## Standard error: how much the estimate itself wobbles

The **standard error** (SE) is the standard deviation of that sampling distribution, it quantifies how much your sample mean would bounce around if you kept re-sampling. It's calculated as SE = s / √n, where s is your sample's standard deviation and n is the sample size.

Worked example: suppose your 500 sampled sessions have a standard deviation of 4.5 minutes. SE = 4.5 / √500 = 4.5 / 22.36 ≈ 0.20 minutes. That's small relative to the 4.5-minute spread of individual sessions, which makes sense, averaging over 500 people smooths out a lot of individual noise. Now suppose you only had 20 users sampled with the same 4.5-minute standard deviation: SE = 4.5 / √20 = 4.5 / 4.47 ≈ 1.01 minutes, five times larger. This square-root relationship is the single most important fact about standard error: precision improves with sample size, but with steeply diminishing returns, going from 20 to 500 shrank SE by 5x, but going from 500 to 2,000 (4x the sample) only shrinks it by another 2x.

## Building the interval

A **confidence interval** combines your point estimate with the standard error to express a plausible range for the true value. For a 95% confidence interval on a mean with a reasonably large sample, the formula is: sample mean ± 1.96 × SE (1.96 comes from the normal distribution, it's the number of standard deviations that captures the middle 95% of a bell curve).

Worked example, continued: sample mean of 8.1 minutes, SE of 0.20. The 95% confidence interval is 8.1 ± 1.96×0.20 = 8.1 ± 0.392, or roughly (7.71, 8.49) minutes. Report this, not just "8.1 minutes," because the interval communicates how much uncertainty is baked into that estimate. With the smaller 20-user sample (SE ≈ 1.01), the same 8.1-minute mean yields a 95% interval of 8.1 ± 1.96×1.01 ≈ (6.12, 10.08), nearly 4 minutes wide instead of 0.78 minutes. Same point estimate, wildly different confidence depending on sample size, which is exactly the information a bare average hides.

## What "95% confidence" actually promises

This is the part that trips people up, including experienced analysts. A 95% confidence interval does **not** mean "there's a 95% probability the true value falls in this specific range." The true average session length is some fixed number, it's not random, it either is or isn't inside (7.71, 8.49). What "95% confidence" actually describes is the *method*: if you repeated this entire sampling-and-interval-construction process many times, about 95% of the resulting intervals would contain the true value, and about 5% would miss it, purely from sampling variability. For any single interval you've already computed, you don't know whether it's one of the 95% that caught the true value or one of the 5% that missed it, you only know the process has a 95% success rate over the long run. This distinction matters when stakeholders want to treat a specific interval as a near-certain bracket. It's a statement about the reliability of the recipe, not a probability attached to the one dish you happened to make.
