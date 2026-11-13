---
title: Descriptive Statistics and Distributions
description: Mean, median, and variance, plus how to read the shape of a distribution to know which summary actually describes your data.
roadmap: ai-data-scientist
node: descriptive-statistics-and-distributions
group: statistics-for-data-science
video:
  provider: youtube
  id: "SzZ6GpcfoQY"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 862
level: beginner
prerequisites: [data-scientist-vs-ml-engineer-vs-analyst]
takeaways:
  - "Mean and median can disagree sharply on skewed data, and the gap between them is itself a signal, not an error to resolve."
  - Variance and standard deviation measure spread in squared and original units respectively, which is why standard deviation is what you report.
  - A distribution's shape, not just its center, tells you whether a single summary number is even meaningful.
  - Outliers pull the mean toward them but barely move the median, which is why median is the safer default for skewed real-world data like income or response time.
check_questions:
  - q: Why can the mean and median of the same dataset be very different numbers?
    a: The mean is pulled toward extreme values because every point, including outliers, contributes its full value to the sum, while the median only cares about which value sits in the middle, so a few very large or very small points can drag the mean away from where most of the data actually sits.
  - q: What does a high variance tell you that the mean alone does not?
    a: It tells you how spread out the values are around the mean, so two datasets can share the same mean but look completely different in practice, one tightly clustered and one scattered widely.
  - q: Why is standard deviation reported more often than variance?
    a: Variance is in squared units, like dollars-squared, which has no intuitive meaning, while standard deviation takes the square root and returns to the original units, making it directly comparable to the data itself.
deeper:
  - label: "The Normal Distribution, Clearly Explained!!!"
    url: "https://www.youtube.com/watch?v=rzFX5NWojp0"
related_formulas: []
---

## Three ways to find the center, and why they disagree

Every dataset has a "typical" value, but there are three common ways to define it, and they only agree when the data is symmetric. The **mean** is the sum of all values divided by the count. The **median** is the middle value when everything is sorted. The **mode** is whichever value appears most often.

Take home prices in a neighborhood: $280k, $310k, $295k, $305k, $320k, and one mansion at $1.2M. The mean is ($280k+$310k+$295k+$305k+$320k+$1,200k)/6 = $452k. The median, the middle of the sorted list, is ($305k+$310k)/2 = $307.5k. Every house but one sold for under $320k, yet the mean says "typical" is $452k. That's not a mistake in arithmetic, it's the mean doing exactly what it's defined to do: incorporate every value's full magnitude into the sum. The median doesn't care how far above or below the middle a point sits, only its rank, so the mansion counts for the same "one vote" as a modest house.

This is why a data scientist checks mean against median before trusting either one. If they're close, the data is roughly symmetric and the mean is a fine summary. If they're far apart, you're looking at a skewed distribution, and the median is usually the more honest "typical" value to report, with the gap itself worth flagging (it tells you there's a long tail worth investigating, not ignoring).

## Measuring spread: variance and standard deviation

Knowing the center isn't enough. Two sales teams can both average $50k per deal while one is wildly inconsistent ($5k, $95k) and the other is steady ($48k, $52k). **Variance** quantifies this: take each value's distance from the mean, square it (so negative and positive distances don't cancel out), and average those squared distances. **Standard deviation** is just the square root of variance, which brings the units back to something interpretable.

Worked example: values 48, 52, 50, 50 have mean 50. Deviations from the mean: -2, 2, 0, 0. Squared: 4, 4, 0, 0. Variance = (4+4+0+0)/4 = 2. Standard deviation = √2 ≈ 1.41. Compare to 5, 95, 50, 50: mean is still 50, deviations are -45, 45, 0, 0, squared are 2025, 2025, 0, 0, variance = 1012.5, standard deviation ≈ 31.8. Same mean, utterly different spread, and standard deviation is the number that makes that difference legible at a glance.

One subtlety: when computing variance from a *sample* meant to estimate a population, you divide by n-1 instead of n (called Bessel's correction). This corrects a slight downward bias that comes from using the sample's own mean, which is pulled slightly toward the sample's data by construction, making the sample look artificially tighter than the true population. For large samples this barely matters; for small ones it does.

## Reading the shape of a distribution

Center and spread describe a distribution, but shape tells you whether those numbers even mean what you think. A **normal distribution** is the symmetric bell curve: mean, median, and mode coincide, and roughly 68% of values fall within one standard deviation of the mean, 95% within two. Heights, measurement errors, and many averaged quantities tend toward this shape.

A **right-skewed** (positive skew) distribution has a long tail stretching toward high values, like the home-price example or household income: most values cluster low, but a few large ones drag the mean above the median. A **left-skewed** distribution is the mirror image, a long tail toward low values, like age at retirement, where a cluster near 65 has a tail of people leaving much earlier. A **bimodal** distribution has two peaks, often a sign you're actually looking at two different populations mixed together, like response times from two different server regions.

Why this matters practically: statistical tests and models that assume normality (including t-tests and linear regression's error terms) can give misleading results on heavily skewed data. Before running a test, plot a histogram. If you see strong skew or two humps, consider a transformation (log transform is the common fix for right-skewed, strictly positive data like prices or durations) or a different summary statistic rather than feeding raw skewed values into a method built for bell curves.

## Putting it together

A single number is never a full picture. Reporting "average order value is $85" while leaving out that the distribution is heavily right-skewed because of a handful of bulk-buyer accounts hides exactly the detail a stakeholder needs to make a good decision. The habit worth building is this: mean, median, standard deviation, and a quick look at the shape, together, every time, before you summarize a dataset to anyone else.
