---
title: The Data Science Lifecycle
description: "The ask, data, analysis, decision, and impact loop that structures a real data science project from start to finish."
roadmap: ai-data-scientist
node: the-data-science-lifecycle
group: foundations
video:
  provider: youtube
  id: "dze8isgT8gU"
  start: 0
  end: null
creator:
  name: Dave Ebbelaar
  channel_url: https://www.youtube.com/@daveebbelaar
duration_seconds: 1695
level: beginner
prerequisites: [what-is-a-data-scientist]
takeaways:
  - A data science project is a loop, not a straight line — findings from analysis routinely send you back to redefine the original question.
  - Most project failures trace back to a skipped or rushed first step, framing the ask, not to a modeling mistake later on.
  - "A model or analysis only has impact once it changes a decision someone makes; measuring that change is part of the lifecycle, not an afterthought."
  - Data collection and cleaning typically consume more project time than analysis or modeling, and should be budgeted that way up front.
check_questions:
  - q: Why is the data science lifecycle better understood as a loop than a straight pipeline?
    a: Findings at almost any stage can invalidate an earlier assumption — for example, discovering the available data can't answer the original question as framed, which sends you back to reframe the ask rather than pushing forward with a worse version of it.
  - q: What's the most common reason a data science project fails, according to the lifecycle framing?
    a: A poorly framed initial question — one that's vague, unmeasurable, or not actually tied to a decision anyone will make — dooms every later step, no matter how good the modeling work is.
  - q: How do you know a data science project actually succeeded?
    a: Not when the model is accurate or the report is delivered, but when it measurably changed a real decision or outcome, which is why tracking the decision and its impact is a formal stage of the lifecycle, not a nice-to-have.
deeper:
  - label: "6 must-have steps for ANY data science project (CRISP-DM explained)"
    url: "https://www.youtube.com/watch?v=jB8I-wKbWYs"
related_formulas: []
---

## Why a project needs a shape at all

Without a structure, a data science project tends to collapse into "get some data and see what's interesting," which produces analyses that are technically fine and practically useless. The lifecycle is a discipline for keeping the work pointed at an actual decision from the first day to the last. It has five stages: the ask, the data, the analysis or model, the decision, and the impact. Frameworks like CRISP-DM break this into more granular steps, but the five-stage version captures the part that matters most for a beginner: this is a loop, and most of the real skill is in resisting the urge to skip to the middle.

## Stage one: the ask

Every project starts with someone wanting something, usually phrased vaguely: "find out why churn is up," "help us understand our users better," "can we predict X." The data scientist's first job is to turn that into a specific, measurable question tied to a decision someone will actually make. "Help us understand our users better" isn't answerable. "Which three signals, in the first seven days after signup, best predict whether a user upgrades to paid within 60 days" is. The difference matters because the second version tells you what data you need, what "done" looks like, and who will use the answer and how. Skipping this stage, and jumping straight to "let's look at the data," is the single most common way these projects go wrong: you end up with a technically interesting finding that doesn't connect to anything anyone was going to decide. A good way to pressure-test an ask is to ask the requester: "if the answer came back as X, what would you do differently?" If there's no answer, the question isn't ready yet.

## Stage two: the data

Once the question is pinned down, you need data that can actually answer it, and this stage is where most project time genuinely goes. Three things commonly go wrong here. First, the data you need doesn't exist yet, because nobody was logging it, which means the real first deliverable is instrumentation, not analysis. Second, the data exists but is scattered: usage events in one system, billing status in another, support tickets in a third, each keyed differently, and joining them correctly is its own project. Third, the data has quality problems that are invisible until you look closely: a "signup_date" field that's actually the date the record was last edited, duplicate user rows from a migration, or a sample that systematically excludes a group (free users who churned before ever being billed, for instance). None of this is a detour from the real work; for the ask about trial-to-paid conversion, building a clean table with one row per user and correctly computed day-7 features might take three weeks against two days of actual modeling.

## Stage three: analysis or model

This is the stage most people picture when they think of data science, and it's often the shortest, in good projects, relative to the stages around it. The right level of sophistication depends entirely on the ask. If the question is "did feature X change behavior," a well-designed comparison of before-and-after cohorts, or better, an A/B test, answers it without any machine learning. If the question is "which users are likely to churn next month, ranked," you need a predictive model, but even then the right starting point is usually the simplest one that could plausibly work: a baseline rate, then logistic regression, and only escalating in complexity if there's a measured accuracy gap that justifies it. Jumping straight to a complex model is a classic beginner mistake, because a complex model trained on poorly understood data produces confident, wrong answers, and the complexity makes those wrong answers harder to debug.

## Stage four: the decision, and stage five: the impact

An analysis that sits in a report nobody reads has not completed the lifecycle. The decision stage is where the finding gets translated into a specific recommendation, delivered in a form the decision-maker can act on: "target these 5,000 users with a retention email, projected to save $80,000 in churned revenue." This is also usually where you discover whether stage one was done properly; if the decision-maker can't connect the finding back to something they can do, the original ask was framed wrong, and you loop back.

The final stage, impact, is the one most frequently skipped, and it's the one that actually proves the work mattered. Did the retention email campaign, built on the model's targeting, actually reduce churn in the group it was sent to, measured against a holdout group that didn't get it? Closing this loop is what separates data science from data storytelling: without it, you never actually learn whether your analysis was right, you just assume it was. Projects that build in a feedback stage from the start, where the real outcome gets compared against the prediction, improve faster over time, because every project becomes a test of the previous one's assumptions rather than a one-off bet.
