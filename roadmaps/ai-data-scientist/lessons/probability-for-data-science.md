---
title: Probability for Data Science
description: Conditional probability, independence, and expectation, the specific probability tools a data scientist actually reaches for day to day.
roadmap: ai-data-scientist
node: probability-for-data-science
group: statistics-for-data-science
video:
  provider: youtube
  id: "_IgyaD7vOOA"
  start: 0
  end: null
creator:
  name: StatQuest with Josh Starmer
  channel_url: https://www.youtube.com/@statquest
duration_seconds: 656
level: beginner
prerequisites: [descriptive-statistics-and-distributions]
takeaways:
  - "Conditional probability, P(A given B), updates your estimate of A once you know B happened, and it is almost always different from P(A) alone."
  - Two events are independent only when knowing one happened changes nothing about the other's probability, a condition that's rarer in real data than intuition suggests.
  - "Expectation is a weighted average of outcomes by their probabilities, and it's the number that tells you what to expect \"on average\" over many repetitions, not what will happen any single time."
  - Confusing P(A given B) with P(B given A) is one of the most common and consequential reasoning errors in applied statistics.
check_questions:
  - q: What is the difference between P(A given B) and P(A and B)?
    a: P(A and B) is the probability both events happen out of all possible outcomes, while P(A given B) restricts attention to only the cases where B already happened and asks what fraction of those also have A, which is why conditioning shrinks the universe you're measuring against.
  - q: Why does testing positive for a rare disease not mean you probably have it?
    a: Because the test's false positive rate applies to the much larger pool of healthy people, so even a small false-positive percentage of a huge healthy population can outnumber the true positives from a tiny diseased population, making most positive results false alarms.
  - q: How is expected value different from the most likely single outcome?
    a: Expected value is the probability-weighted average across every possible outcome, which can be a number that never actually occurs, like an expected 2.5 customers per minute, while the most likely single outcome is just whichever specific result has the highest individual probability.
deeper:
  - label: "Expected Values, Main Ideas!!!"
    url: "https://www.youtube.com/watch?v=KLs_7b7SKi4"
  - label: "Conditional probability and independence"
    url: "https://www.youtube.com/watch?v=pIfpHdGVwLU"
related_formulas: []
---

## Conditional probability: the tool you use constantly without naming it

Most probability questions a data scientist actually faces aren't "what's the chance of X" in isolation, they're "given that I already know Y, what's the chance of X." That's conditional probability, written P(A|B), read "the probability of A given B." Formally, P(A|B) = P(A and B) / P(B): out of all the cases where B happened, what fraction also have A.

Worked example: say 40% of your website's visitors are on mobile, and among mobile visitors, 8% complete a purchase, versus 15% among desktop visitors. If someone asks "what's the probability a random visitor converts," you'd need to combine these: P(convert) = P(mobile)×P(convert|mobile) + P(desktop)×P(convert|desktop) = 0.40×0.08 + 0.60×0.15 = 0.032 + 0.09 = 0.122, or 12.2% overall. Notice that neither the 8% nor the 15% alone describes the full population; conditioning on device type and then recombining is what gets you the real answer. This pattern, breaking a question into conditional pieces and weighting by how likely each condition is, is the backbone of a huge amount of applied data work.

## Independence, and why it's easy to assume wrongly

Two events A and B are **independent** if P(A|B) = P(A), meaning knowing B happened tells you nothing new about A. Coin flips are independent: knowing the first flip was heads tells you nothing about the second. But a lot of real-world variables that look unrelated aren't. Whether a customer churns and whether they contacted support last month are usually not independent. Support contact is a strong positive signal of churn risk, which is exactly why support-ticket volume is a common churn-model feature.

The practical risk is assuming independence to simplify a calculation (it makes math and modeling much easier) when the variables are actually correlated. A classic failure mode: treating daily stock returns as independent when they exhibit volatility clustering, a volatile day is more likely to be followed by another volatile day. Models and risk estimates built on a false independence assumption underestimate how extreme outcomes can get, because correlated bad events can compound instead of average out.

## The base rate trap: why P(A|B) ≠ P(B|A)

One of the most consequential mistakes in applied probability is swapping P(A|B) for P(B|A) as though they were the same thing. They are related by Bayes' rule but are generally very different numbers, and the gap matters most when one event is rare.

Worked example: a disease affects 1 in 1,000 people (0.1% prevalence). A test is 99% accurate: it correctly flags 99% of people who have the disease (true positive rate) and correctly clears 99% of people who don't (true negative rate, meaning a 1% false positive rate). You test positive. What's the probability you actually have the disease? Out of 100,000 people: 100 have the disease, and the test catches about 99 of them. Of the 99,900 healthy people, 1% (about 999) test positive anyway, falsely. So out of roughly 1,098 total positive results (99 + 999), only 99 are real, about 9%. A 99%-accurate test on a rare condition still means a positive result is more likely a false alarm than a true one. This is the gap between P(positive|disease), which is 99%, and P(disease|positive), which is only about 9%, and confusing the two leads people to wildly overestimate what a positive test means.

## Expectation: the long-run average that guides decisions

**Expected value** is the sum of each possible outcome multiplied by its probability. It answers "what happens on average, if I could repeat this many times," which is exactly the question behind pricing, risk, and resource-allocation decisions.

Worked example: a marketing campaign costs $10,000. There's a 30% chance it generates $50,000 in revenue, a 50% chance it generates $8,000, and a 20% chance it generates nothing. Expected revenue = 0.30×$50,000 + 0.50×$8,000 + 0.20×$0 = $15,000 + $4,000 + $0 = $19,000. Expected profit = $19,000 - $10,000 = $9,000. That doesn't mean this specific campaign will net $9,000, it almost certainly won't hit that exact number, it means that across many similar campaigns run under these odds, $9,000 is the average outcome you'd converge toward. This distinction, between what you expect on average and what any single instance will actually produce, is central to interpreting A/B test results, model confidence scores, and any forecast that reports a single number instead of a distribution.
