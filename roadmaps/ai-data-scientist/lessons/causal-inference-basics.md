---
title: Causal Inference Basics
description: Why correlation isn't causation, how confounders fool regressions, and the core toolkit for recovering causal effects without a lab experiment.
roadmap: ai-data-scientist
node: causal-inference-basics
group: econometrics-and-experimentation
video:
  provider: youtube
  id: "GtV-VYdNt_g"
  start: 0
  end: null
creator:
  name: CrashCourse
  channel_url: https://www.youtube.com/@crashcourse
duration_seconds: 737
level: intermediate
prerequisites: [regression-for-inference]
takeaways:
  - A correlation between two variables is consistent with X causing Y, Y causing X, a third variable causing both, or pure chance, and data alone cannot tell you which.
  - Randomization is the gold standard for causal claims because it breaks the link between treatment and every confounder, observed or not, by construction.
  - "When randomization isn't possible, instrumental variables and difference-in-differences are two ways researchers approximate it using the structure of naturally occurring data."
  - Difference-in-differences relies on a parallel trends assumption that is easy to state and genuinely hard to verify.
check_questions:
  - q: Why can't a strong correlation between two variables tell you which one causes the other, or whether either does?
    a: A correlation is equally consistent with X causing Y, Y causing X, an unmeasured third variable driving both, or coincidence, so the number alone carries no information about direction or mechanism.
  - q: What specifically does randomization accomplish that a carefully controlled regression cannot?
    a: Random assignment makes treatment statistically independent of every other factor that affects the outcome, including ones the researcher never measured or thought to control for, whereas a regression can only control for confounders it explicitly includes.
  - q: What assumption does a difference-in-differences design require, and why is it hard to check?
    a: It requires that the treatment and control groups would have moved in parallel over time absent the treatment, which is a claim about a counterfactual that, by definition, was never observed.
deeper:
  - label: "Introduction to Instrumental Variables (IV)"
    url: "https://www.youtube.com/watch?v=eoJUPd6104Q"
  - label: "Introduction to Differences-in-Differences"
    url: "https://www.youtube.com/watch?v=eiffOVbYvNc"
related_formulas: []
---

## The gap that correlation can't close

Cities with more firefighters tend to have more fire damage. Taken at face value, that correlation suggests firefighters cause damage, which is absurd — the real story is that bigger fires draw more firefighters and also cause more damage, so fire severity drives both. This is the standard illustration of a general fact: a correlation between X and Y is compatible with four distinct underlying stories — X causes Y, Y causes X, some third variable Z causes both, or the pattern is coincidence in a finite sample. No amount of staring at the correlation coefficient tells you which story is true. The previous lesson showed how omitted variables distort regression coefficients; this lesson is about what you can actually do when you want to make a causal claim despite that limitation, because in a data science job, "we can't know" is rarely an acceptable final answer when a decision is riding on it.

## Confounders, named properly

A confounder is a variable that influences both the thing you think is a cause and the thing you think is an effect. In the firefighter example, fire severity is the confounder. In a business setting, suppose you notice that customers who open your app's push notifications spend more money, and you're tempted to conclude notifications drive spending. But notification opens and spending might both be driven by a confounder: overall engagement. Highly engaged users open everything and also buy more; low-engagement users do neither. Send more notifications to everyone and you might see no revenue lift at all, because the correlation was never causal. The practical skill is building a habit of asking, for any observed relationship you're tempted to act on, "what third thing could plausibly move both of these together?" before trusting it.

## Randomization: breaking the link on purpose

The cleanest solution to confounding is to make the "cause" independent of everything else by construction, which is exactly what random assignment does. If you randomly assign half of users to receive a notification and half not to, then by the law of large numbers the two groups will have statistically similar engagement levels, similar purchase histories, similar everything, known and unknown. Any resulting difference in spending between the groups can then be attributed to the notification itself, because randomization severed the connection between the treatment and every possible confounder simultaneously, including the ones nobody thought to measure. This is the logical foundation behind A/B testing, which the next lesson covers in operational detail, and it's why a well-run randomized experiment is treated as stronger evidence than even a very careful observational regression.

## When you can't randomize: two workarounds

Often you can't run an experiment — you can't randomly assign which countries adopt a policy or which students attend which school. Two techniques let researchers approximate randomization's logic using data that already exists. An instrumental variable is a factor that affects your outcome only indirectly, through its effect on the treatment, and is otherwise unrelated to the confounders. Classic example: researchers studying the effect of education on earnings have used distance to the nearest college as an instrument, since living closer makes you more likely to attend college but, plausibly, doesn't directly affect your earnings potential otherwise. If the instrument is valid, the variation it creates in the treatment is "as good as random," letting you isolate a causal effect even without an experiment. Difference-in-differences takes a different angle: it compares the before-and-after change in an outcome for a group that received some treatment against the before-and-after change for a group that didn't, and attributes the gap between those two changes to the treatment. If a state raises its minimum wage and a neighboring state doesn't, comparing employment trends in both states before and after the change can isolate the policy's effect, as long as the two states would have trended similarly absent the policy.

## The assumption that makes or breaks diff-in-diff

That last clause, "as long as the two states would have trended similarly," is the parallel trends assumption, and it's the single most important thing to interrogate in any difference-in-differences analysis. It's a statement about a counterfactual world that never happened — what would have occurred in the treated state if the policy hadn't passed — so it can never be directly verified, only argued for. The standard check is to look at the pre-treatment trends: if both groups were moving in lockstep for several periods before the treatment, it's more plausible they'd have continued in lockstep without it. But a good pre-trend is suggestive, not proof, and this is exactly the kind of judgment call that separates a causal claim worth acting on from one that only looks rigorous because it came with a regression table attached.
