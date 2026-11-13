---
title: Regression for Inference
description: Using linear regression to test hypotheses about relationships, not just to predict, and why standard errors matter more than the coefficient itself.
roadmap: ai-data-scientist
node: regression-for-inference
group: econometrics-and-experimentation
video:
  provider: youtube
  id: "fZV4ntLEPlU"
  start: 0
  end: null
creator:
  name: Ben Lambert
  channel_url: https://www.youtube.com/@SpartacanUsuals
duration_seconds: 522
level: intermediate
prerequisites: [bayesian-thinking-for-data-scientists]
takeaways:
  - A regression coefficient is a point estimate of a relationship in the population, and without its standard error it tells you almost nothing about how much to trust it.
  - "A small p-value answers one narrow question: is this coefficient distinguishable from zero? It does not tell you the effect is large, important, or causal."
  - Confidence intervals communicate what a single p-value hides — the plausible range of effect sizes, which is usually the number a decision actually needs.
  - Omitting a variable that affects both the outcome and an included predictor biases every coefficient in the model, often silently.
check_questions:
  - q: Why is a coefficient's standard error as important as the coefficient itself?
    a: The coefficient is one estimate drawn from one sample, and the standard error tells you how much that estimate would bounce around if you redrew the sample, so a coefficient with a huge standard error could easily be noise even if it looks large.
  - q: "What does a p-value of 0.03 on a coefficient actually tell you, and what does it not tell you?"
    a: It tells you the estimate is unlikely to arise by chance alone if the true effect were zero, nothing more; it does not tell you the effect is large, practically meaningful, or that the predictor causes the outcome.
  - q: Why can adding or leaving out a variable change the coefficient on a variable you already had in the model?
    a: If the omitted variable is correlated with both the outcome and a predictor already in the model, its influence gets absorbed into that predictor's coefficient, biasing it away from the true relationship.
deeper:
  - label: "Inference on the Slope (The Formulas)"
    url: "https://www.youtube.com/watch?v=THzckPB7E8Q"
related_formulas: []
---

## Two different jobs for the same equation

Linear regression shows up in two very different jobs that happen to use the same math. In prediction, the goal is a number that's as close as possible to the real outcome for a new observation, and you don't care much why the model works. In inference, the goal is the opposite: you care intensely about the relationship a specific coefficient represents, and the fitted line itself is almost beside the point. A marketing team predicting next month's revenue wants the forecast. An economist asking whether an extra year of schooling raises wages, holding experience fixed, wants the coefficient on schooling and a defensible statement about how confident to be in it. This lesson is about the second job: how to read a regression output like an argument about the world rather than a prediction engine.

## What a coefficient actually estimates

Suppose you regress hourly wage on years of education and years of experience. The output gives you a coefficient on education, say 2.1. That number claims: holding experience fixed, each additional year of education is associated with $2.10 more in hourly wage, on average, across the data you have. Two things matter immediately. First, "holding experience fixed" is doing real work — this is the entire reason to use multiple regression instead of just comparing average wages across education levels, which would mix the effect of education with the effect of experience riding along with it. Second, "on average, across the data you have" is an estimate of an unknown population relationship, not the relationship itself. If you drew a different sample of workers, you'd get a different number, maybe 1.8 or 2.4. The standard error quantifies how much that number would typically move across different samples, and it's computed from the spread of the residuals and the amount of variation in the predictor itself: noisier outcomes or a predictor that barely varies in your sample both inflate the standard error.

## From standard error to a real statement of confidence

Once you have a coefficient and its standard error, you can build a t-statistic (the coefficient divided by its standard error) and from that a p-value and a confidence interval. The p-value answers a narrow, specific question: if the true coefficient were actually zero, how unusual would an estimate this large be? A p-value of 0.03 means an estimate this far from zero would happen about 3% of the time by chance alone if there were truly no relationship. That's useful for ruling out noise, but it says nothing about size. A coefficient of 0.02 with a tiny standard error can be "highly significant" while being economically irrelevant, and a coefficient of 8.0 with a wide standard error can fail to clear significance while still being the more important number to act on if it's real. This is why a 95% confidence interval — roughly the coefficient plus or minus two standard errors — is often more useful to report than the p-value alone: an interval of [1.9, 2.3] tells a decision-maker what a single star next to a p-value cannot, namely the plausible range of true effect sizes.

## Why interpretation breaks before significance does

The more dangerous failure mode in inferential regression isn't a misread p-value, it's omitted variable bias, and it happens silently. Imagine regressing a city's ice cream sales on its crime rate and finding a positive, significant coefficient. Does ice cream cause crime? Both are driven by a third variable, temperature: hot days bring out more ice cream buyers and more people outdoors, which correlates with more crime. Leave temperature out of the model and its effect doesn't disappear, it gets absorbed into whichever included variable correlates with it, distorting that coefficient's meaning. The fix isn't a statistical trick, it's domain knowledge: before trusting a coefficient, ask what else plausibly affects the outcome and is also correlated with your predictor of interest, and either control for it or treat your coefficient as suggestive rather than settled. This is the bridge to the next lesson's topic, causal inference, which exists specifically because regression coefficients, even well-estimated ones, describe correlations that require much more care before being read as causes.

## Reading an output table like it matters

In practice, reading a regression table for inference means working through it in a specific order: look at the coefficient's sign and magnitude first (does it make substantive sense), then its standard error and confidence interval (how precisely is it estimated), then consider what's missing from the model that could be confounding it, and only then glance at the p-value as a final sanity check rather than the headline. A model with a dozen significant stars and no plausible story for why its controls were chosen is weaker evidence than a model with three carefully chosen variables and one coefficient reported honestly with a wide interval.
