---
title: "Data Scientist vs. ML Engineer vs. Data Analyst"
description: "How these three roles actually differ in scope, deliverables, and skills at a typical company, not just by title."
roadmap: ai-data-scientist
node: data-scientist-vs-ml-engineer-vs-analyst
group: foundations
video:
  provider: youtube
  id: "5BzlXxmEP60"
  start: 0
  end: null
creator:
  name: Accredian
  channel_url: https://www.youtube.com/@Accredian
duration_seconds: 336
level: beginner
prerequisites: [math-and-stats-you-actually-need]
takeaways:
  - "The three roles split primarily by deliverable: an analyst delivers an answer or a dashboard, a data scientist delivers an insight or a model, an ML engineer delivers a running production system."
  - "Analysts work with tools built for them (SQL, BI dashboards); data scientists build the analysis; ML engineers build and operate the infrastructure the analysis runs on at scale."
  - Software engineering skill, testing, deployment, monitoring, system design, is the main differentiator between a data scientist and an ML engineer, not math.
  - At smaller companies these roles blur into one person; at larger companies the boundaries are explicit and often contractual between teams.
check_questions:
  - q: What's the clearest way to tell a data analyst's job apart from a data scientist's?
    a: An analyst typically answers a known, well-scoped question using existing data and tools, producing a report or dashboard, while a data scientist is more often asked to define the question itself and build something new, like a model, to answer it.
  - q: What specifically separates a data scientist from a machine learning engineer?
    a: "Software engineering depth: a data scientist typically builds a model in a notebook and proves it works, while an ML engineer takes that model (or builds one directly) and makes it run reliably in production, handling deployment, latency, scaling, and monitoring."
  - q: Why do these three roles often blur together at small companies?
    a: A startup usually can't afford three specialists, so one person ends up writing the SQL, building the model, and deploying it, meaning the title "data scientist" at a 10-person startup can mean all three jobs combined.
deeper:
  - label: "Data Scientist vs Machine Learning Engineer | DS vs ML"
    url: "https://www.youtube.com/watch?v=rLvpY2v9hWQ"
related_formulas: []
---

## Three roles, one confusing job market

Job postings routinely use "data scientist," "machine learning engineer," and "data analyst" inconsistently, sometimes for the same job, which makes the roles feel arbitrary. They aren't. At a company organized with any maturity, the three roles split along a clear line: what question is being asked, and what gets delivered at the end. Understanding that line helps you both pick which path fits you and read a job posting for what the work will actually be, regardless of what title it's stamped with.

## The data analyst: answering known questions, fast and reliably

A data analyst's core job is to take a well-understood business question, "what were sales by region last quarter," "which marketing channel has the best conversion rate," and answer it using existing data and tools: SQL queries, spreadsheets, BI dashboards like Tableau or Looker. The skill here is less about inventing a method and more about speed, accuracy, and communication: knowing the data model well enough to write a correct query quickly, catching when a number looks wrong before it goes in a slide, and building a dashboard stakeholders can self-serve from instead of asking for a new report every week. Analysts generally don't build predictive models or write production code; their output is a correct answer or a reusable reporting tool, used by someone else to make a decision. A strong analyst can often answer in an afternoon a question that would take a data scientist a week, because the analyst isn't inventing the methodology, they're applying a known one to known data.

## The data scientist: defining and answering new questions

A data scientist's job starts one level up: the question often isn't fully defined yet, and part of the job is figuring out what's actually worth measuring and how. Where an analyst might be asked "what's our churn rate," a data scientist is more often asked "why are we losing customers, and can we predict who's at risk," which requires building something new, a cohort analysis framework, a churn model, an experiment design, rather than querying something that already exists. The deliverable is usually an insight, a recommendation, or a working model proven out in a notebook or a controlled test, not a production system serving live traffic. This is the role most associated with the mix of statistics, light programming, and business judgment.

## The ML engineer: making it run, reliably, at scale

A machine learning engineer's job begins roughly where a data scientist's modeling work ends, or sometimes earlier if the ML engineer builds the model too, and extends into territory that's much closer to software engineering than statistics: taking a model and making it run correctly and efficiently in production. That means writing code that handles real-time or batch inference at scale, building the pipeline that retrains the model on fresh data automatically, monitoring for when the model's accuracy quietly degrades because real-world data drifted from training data, and designing for failure (what happens when the model service times out, or a malformed input arrives). This role draws much more on software engineering fundamentals, testing, version control discipline, system design, deployment, than on advanced statistics. A data scientist who built a 94%-accurate churn model in a Jupyter notebook has not yet done the ML engineer's job; turning that notebook into a service that scores every user nightly, alerts when its inputs look unusual, and can be rolled back safely if a bad model gets deployed, is a separate and substantial body of work.

## A worked comparison

Take one question: "reduce cart abandonment on an e-commerce site." An analyst would pull the abandonment rate by device, time of day, and cart value, and hand over a dashboard showing where the problem concentrates. A data scientist would take that further: build a model predicting which in-progress carts are likely to be abandoned, test whether a targeted discount email changes that outcome, and recommend who should receive it. An ML engineer would take the winning model and discount-trigger logic and build the system that scores every live cart in real time, sends the trigger through the email platform, logs outcomes for retraining, and pages someone if the scoring service's latency spikes. All three are necessary; none of them is a subset of the others, they're a sequence with different skills at each stage.

## Why the lines blur at smaller companies

None of this means the roles are rigid in practice. At a startup with one data hire, that person often does all three jobs across different hours of the same week: analyst-style reporting on Monday, a prediction model on Wednesday, and deploying it themselves on Friday because there's no separate ML engineering team yet. The three-role split is a description of how the work decomposes once a company is large enough to specialize, not a rule about what any one person's job must look like. Knowing the split still matters early in a career, because it tells you which skills to build depending on which part of that sequence you want to spend most of your time in.
