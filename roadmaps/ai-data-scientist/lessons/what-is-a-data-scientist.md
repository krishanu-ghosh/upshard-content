---
title: What Is a Data Scientist?
description: "What a data scientist actually does day to day, and how the role differs from \"someone who's good with data.\""
roadmap: ai-data-scientist
node: what-is-a-data-scientist
group: foundations
video:
  provider: youtube
  id: "MvV2xY_u1rw"
  start: 0
  end: null
creator:
  name: Data with Baraa
  channel_url: https://www.youtube.com/@DataWithBaraa
duration_seconds: 938
level: beginner
prerequisites: []
takeaways:
  - A data scientist's real job is answering a specific business question with evidence, not running every algorithm available.
  - "Most of the actual work is unglamorous: finding data, cleaning it, and deciding what to measure, long before any modeling happens."
  - The title covers a wide range of jobs; a data scientist at a 20-person startup and one at a bank do genuinely different work day to day.
  - Communicating a result in terms a non-technical stakeholder can act on is as much a core skill as writing the analysis itself.
check_questions:
  - q: Why is "someone who's good with data" not a useful definition of a data scientist?
    a: "It describes a skill, not a job: data scientists are hired to answer specific decisions with evidence, and most of their time goes into tasks like data cleaning, defining metrics, and communicating results, not algorithm selection."
  - q: What makes data science different from a data analyst role, in practice?
    a: A data analyst typically answers a known question with existing tools and dashboards, while a data scientist is more often asked to figure out what question to even ask, build something new to answer it, and sometimes ship that answer as a running system.
  - q: Why does a data scientist's day-to-day work vary so much between companies?
    a: Company size and maturity determine what's missing — a startup data scientist often builds the data pipeline themselves and does analyst-style reporting too, while a mature company's data scientist works downstream of dedicated data engineering and analytics teams.
deeper:
  - label: "What it's really like to be a data scientist"
    url: "https://www.youtube.com/watch?v=KjwI5kKHyhY"
related_formulas: []
---

## The job behind the title

"Data scientist" is one of the more inflated titles in tech, partly because the role was invented quickly to describe people doing statistics, programming, and business analysis at once, and partly because companies use it inconsistently. Strip away the hype and the job comes down to one thing: turning messy, real-world data into a decision someone can actually act on. A product manager wants to know why signups dropped last month. A hospital wants to predict which patients are likely to be readmitted. A retailer wants to know how much inventory to order before a holiday. In every case, the data scientist's job is to take a vague, often badly-specified question, turn it into something measurable, go find or build the data needed to measure it, and come back with an answer that holds up under scrutiny.

That last part, "holds up under scrutiny," is what separates the job from just running code. Anyone can fit a model to a dataset and report a number. A data scientist has to know whether that number means anything: whether the sample was biased, whether the correlation is actually causal, whether the model will still work next quarter when the underlying behavior shifts. This is why the role sits at the intersection of statistics, programming, and domain knowledge rather than being purely a programming job. You need the statistics to know when a result is noise. You need the programming to actually get and process the data at scale. And you need enough domain knowledge to know which question is worth asking in the first place.

## What the work actually looks like week to week

The popular image of a data scientist is someone training neural networks all day. The reality, confirmed by every serious survey of the field, is that 60-80% of the job is data preparation: finding where the relevant data lives, figuring out why half the rows have missing values, reconciling two systems that define "active user" differently, and building a dataset that's actually trustworthy enough to analyze. Only after that does the "fun part" start, and even then, a huge fraction of real data science work is exploratory analysis and reporting, not machine learning. A data scientist at an e-commerce company might spend a week figuring out why conversion rates look different across two dashboards before ever building a model. Finding that the two dashboards use different time zones for the cutoff is a data science task, even though it involves no statistics or machine learning at all.

When modeling is the right tool, it's usually simpler than people expect. A churn prediction problem is often solved well by logistic regression with ten carefully chosen features, not a deep neural network. The skill isn't "use the most advanced method," it's "use the simplest method that answers the question reliably, and know when a fancier one is actually justified." A data scientist who reaches for a complex model when a simple one would work just as well, and takes three times as long to build and explain, has made a judgment error, not a technical one.

## Where the role's boundaries actually are

The title "data scientist" gets stretched to cover very different jobs depending on the company. At an early-stage startup, a data scientist might be the only data person on staff: they write the SQL to extract data, build the pipeline that refreshes it, do the executive dashboard, and run the occasional model, because there's no one else to do any of those things. At a large, data-mature company, the same title might mean something much narrower: you receive clean data from a dedicated data engineering team, your dashboards are built by a separate analytics team, and your job is specifically experimentation design and predictive modeling for one product area. Neither version is "more real" than the other, but it means two people with the same job title, hired in the same year, can have almost no overlap in their actual day-to-day tasks. This is worth knowing going in, because it changes what you should ask in an interview: not "do you do data science here," but "what, specifically, does a data scientist spend their time on at this company."

## The skill that gets underrated: explaining the answer

A technically correct analysis that nobody acts on is a failed project. Part of the job, often the part new data scientists are least prepared for, is translating a result into language a stakeholder without a statistics background can use to make a decision. Saying "the model has an AUC of 0.81" means nothing to a marketing director. Saying "this model would let us target the 20% of customers most likely to churn, and catching even a third of them would save about $400,000 a year" is the same result, made usable. Data scientists who can't do this translation step end up producing analyses that sit in a slide deck and change nothing, regardless of how rigorous the underlying work was.
