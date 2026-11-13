---
title: Designing A/B Tests
description: How to size, randomize, and run an A/B test that gives a trustworthy answer, and the peeking trap that invalidates most informal ones.
roadmap: ai-data-scientist
node: designing-ab-tests
group: econometrics-and-experimentation
video:
  provider: youtube
  id: "HEGI5QN3fXE"
  start: 0
  end: null
creator:
  name: CXL
  channel_url: https://www.youtube.com/@CXLdotcom
duration_seconds: 1594
level: intermediate
prerequisites: [confidence-intervals-and-sampling, causal-inference-basics]
takeaways:
  - "A valid A/B test needs its sample size fixed in advance, calculated from the baseline rate, minimum detectable effect, and desired power, before any data comes in."
  - Checking results repeatedly and stopping as soon as you see significance inflates your false-positive rate far above the 5% you think you're getting, a failure called peeking.
  - Running many simultaneous comparisons, such as testing ten metrics or ten segments at once, multiplies your chance of a false positive unless you correct for it.
  - Statistical significance tells you an effect probably isn't zero; it does not by itself tell you the effect is large enough to be worth shipping.
check_questions:
  - q: "Why does a team need to calculate its sample size before starting an A/B test, rather than just running it and checking in periodically?"
    a: The sample size calculation, based on the baseline conversion rate, the smallest effect worth detecting, and the desired statistical power, is what guarantees the test has a fair chance of detecting a real effect and controls the false-positive rate; checking in periodically and stopping early undermines both guarantees.
  - q: What is "peeking," and why does it make a test's reported significance misleading?
    a: Peeking is checking a test's results repeatedly during the run and stopping as soon as the p-value crosses the significance threshold, and it's misleading because the threshold assumes you looked at the data once, so checking many times gives random noise many chances to cross it, inflating the real false-positive rate well above 5%.
  - q: If you test ten different metrics in a single A/B experiment, why is a single result showing p less than 0.05 less convincing than it would be in an isolated test?
    a: Running ten independent comparisons at a 5% significance threshold gives roughly a 40% chance that at least one shows a false positive purely by chance, so an isolated significant result among many tested is much weaker evidence than the same result from one pre-specified comparison.
deeper:
  - label: "Peeking at A/B Tests: Why it matters, and what to do about it"
    url: "https://www.youtube.com/watch?v=6G8F22NSCxk"
  - label: "A/B Testing 101: How to Calculate the Sample Size"
    url: "https://www.youtube.com/watch?v=KC1nwY7YCUE"
related_formulas: []
---

## What an A/B test is actually testing

An A/B test takes the randomization logic from causal inference and operationalizes it for a product decision: randomly split users into a control group seeing the current experience and a treatment group seeing a variant, then compare an outcome metric between them. Because assignment is random, any systematic difference in the outcome can be attributed to the variant rather than to pre-existing differences between the groups — the same logic that makes randomized experiments the strongest form of causal evidence. But "randomly split and compare" hides three design decisions that determine whether the result means anything: how many users you need, how you decide when to stop looking, and how many things you're comparing at once. Get any of the three wrong and you can produce a result that looks statistically clean while being nearly worthless.

## Sizing the test before you run it

Before launching, you need to fix how many users each arm requires, and that number depends on three inputs you choose up front: the baseline rate (say, your current checkout conversion is 4%), the minimum detectable effect you actually care about (do you want to catch a move to 4.4%, or only a move to 5%?), and the statistical power you want, conventionally 80%, meaning an 80% chance of detecting the effect if it's really there. Smaller effects and higher power both demand more users — wanting to detect a move from 4% to 4.2% might require tens of thousands of users per arm, while a move from 4% to 6% might need a few thousand. The sample size calculator isn't a formality; it's the thing that tells you honestly whether your traffic can even answer your question in a reasonable timeframe, and it's far better to learn that before running a four-week test than after.

## The peeking trap

Here is the single most common way real A/B tests go wrong. A team launches a test needing 20,000 users per arm, but checks the dashboard daily. On day 4, with only a fraction of the planned sample, the p-value dips below 0.05 and someone calls it a win and ships. The problem: a p-value threshold of 0.05 is calibrated for looking at the data exactly once, at the pre-planned sample size. If you check every day for twenty days, you give random noise twenty separate chances to randomly dip below the threshold, even when there's truly no effect. Simulations of this scenario show the real false-positive rate under daily peeking-and-stopping can climb well above 30%, several times the nominal 5%. The fix isn't "never look" — looking is fine for monitoring that nothing is broken — the fix is deciding in advance not to stop and declare a winner until the pre-calculated sample size is reached, or using a sequential testing method specifically designed to allow valid early stopping, which adjusts the significance threshold at each look to compensate.

## The multiple-comparisons trap

A sibling problem shows up when a test measures many things at once: say a test tracks ten different metrics, or evaluates results separately for five user segments. At a 5% significance threshold, if none of the ten comparisons reflect a real effect, the chance that at least one shows up significant by pure chance is roughly 1 minus 0.95 to the tenth power, about 40%. Teams fall into this trap by running one test, finding nothing on the primary metric, then slicing the data by country, device, and signup cohort until something turns significant, and reporting that slice as the finding. That slice was found by search, not hypothesis, and it will usually fail to replicate. The fix is to decide your primary metric before the test starts and treat any other significant result found by slicing as a hypothesis for a new, dedicated test, not as a finding to ship on its own. When you do need multiple planned comparisons, corrections like Bonferroni (dividing your significance threshold by the number of comparisons) keep your overall false-positive rate honest.

## Significance is not the same as "ship it"

Even a cleanly run test that avoids both traps only answers one question: is this effect probably not zero? It doesn't answer whether the effect is big enough to matter. A test with 100,000 users per arm can detect a genuinely real but tiny 0.1 percentage point lift with high confidence — statistically significant, but possibly not worth the engineering cost or the risk of added complexity. This is why the minimum detectable effect chosen during sizing matters twice: once to size the test correctly, and again at the end, to remind you that "significant" was defined relative to an effect size you decided in advance was worth caring about.
