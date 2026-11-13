---
title: "SLIs, SLOs, and Error Budgets"
description: How SLIs, SLOs, and SLAs turn "how reliable should this be" into a number, and how an error budget governs release velocity.
roadmap: system-design
node: slis-slos-and-error-budgets
group: observability-and-security-at-scale
video:
  provider: youtube
  id: "Dfnbw5dJQ5I"
  start: 0
  end: null
creator:
  name: DevOpsDays Chicago
  channel_url: https://www.youtube.com/@DevOpsDaysChicago
duration_seconds: 1859
level: intermediate
prerequisites:
  - monitoring-logging-and-alerting
  - measuring-availability-the-nines
takeaways:
  - An SLI is a directly measured number, like the percentage of requests served under 300ms or the percentage that returned a non-error status.
  - An SLO is a target drawn on top of an SLI, like "99.9% of requests succeed, measured over a rolling 30 days" — it's internal and it's what teams actually operate against.
  - An SLA is the externally-facing promise, usually with financial or contractual consequences for missing it, and it should be looser than the internal SLO so you breach your own target before you breach your customer's.
  - An error budget is simply 1 minus the SLO, turned into a concrete allowance — a 99.9% SLO over 30 days allows about 43 minutes of failure, however that failure gets spent.
  - Once the error budget for a period is exhausted, the standard policy is to freeze new feature releases and redirect effort to stability until the budget recovers.
  - A 100% reliability target is usually the wrong goal — it eliminates any room for shipping risky changes, and most users can't tell the difference between 99.9% and 100% anyway.
check_questions:
  - q: What's the difference between an SLI and an SLO?
    a: An SLI is the raw measured number, such as the percentage of requests that succeeded in the last five minutes, while an SLO is the target you set for that number over a longer window, such as 99.9% success measured over 30 days.
  - q: Why should a company's internal SLO usually be stricter than its external SLA?
    a: The SLA carries contractual or financial consequences for the company if it's breached, so the internal SLO needs tighter margin to trigger a response and get the system back in line before the looser, customer-facing SLA is ever actually at risk.
  - q: If a team has burned through this month's entire error budget, what should happen next?
    a: Under the standard error-budget policy, the team stops shipping new features and shifts its effort to reliability work — bug fixes, hardening, reducing risk — until reliability recovers enough to reopen budget for new releases.
deeper:
  - label: "Scaling Reliability at Twitter: So You Want to Add a 9"
    url: "https://www.youtube.com/watch?v=hYu13kBenjE"
  - label: "The Tyranny of the SLA"
    url: "https://www.youtube.com/watch?v=4cPqLuIXBnw"
related_formulas: []
---

## Turning "how reliable should this be" into an actual number

Every system has an implicit reliability expectation, and left implicit, it causes two kinds of fights. Engineers want time to ship features and take reasonable risks; whoever's accountable for the product wants nothing to ever break. Without a shared, explicit number to point at, both sides are arguing from vibes. Site reliability engineering's answer — now standard practice well beyond Google, where the terminology originated — is to make reliability a measured target instead of a feeling, using three related but distinct pieces: the SLI, the SLO, and the SLA.

A **service level indicator (SLI)** is a directly measured quantity: the percentage of HTTP requests in the last five minutes that returned a 2xx or 3xx status, or the percentage of requests served in under 300ms, or the percentage of writes that were durably committed. It's just a number coming off your metrics pipeline — the same metrics telemetry from the previous lesson, pointed at a specific question.

A **service level objective (SLO)** is a target set on top of an SLI, scoped to a time window: "99.9% of requests succeed, measured over a rolling 30 days." The SLO is what a team actually operates against day to day — it's internal, it's the thing dashboards are built around, and it's specific enough that anyone can check whether it's currently being met.

A **service level agreement (SLA)** is the externally-facing version — a promise made to customers, usually with real consequences (service credits, refunds, contract penalties) attached to missing it. The important design rule is that the SLA should be looser than the internal SLO. If your SLA promises 99.5% and your internal SLO targets 99.9%, you'll notice you're in trouble and react while you're still comfortably inside the customer-facing promise, rather than finding out you're in breach of a contract at the same moment you find out you have a problem.

## The error budget: the gap you're allowed to spend

Once there's an SLO, the gap between it and perfect reliability has a name: the **error budget**. If the SLO is 99.9% availability over 30 days, the error budget is the remaining 0.1% — which works out to roughly 43 minutes of acceptable downtime (or failed requests, depending on what the SLI measures) across that month. That's not a hypothetical number; it's a budget in the literal sense, something you're allowed to spend, and the whole point of framing it this way is that spending it is not automatically bad. A risky deploy, an experimental feature, an infrastructure migration — all of these carry some chance of an incident, and the error budget is explicit permission to take that chance, as long as you haven't already spent the month's allowance.

This reframes an argument that used to be purely subjective ("is it safe to ship this?") into an arithmetic one ("how much budget is left, and how much does this deploy put at risk?"). It also explains why chasing 100% reliability is usually a mistake rather than a virtue: a 100% target has zero error budget, which means zero room to ship anything that could possibly fail — including routine deploys — and most users genuinely cannot perceive the difference between 99.9% and 100% uptime anyway. The economically sensible target is "reliable enough that users don't notice or care," not "perfect."

## When the budget runs out, releases slow down

The error budget only does useful work if burning it has a consequence, and the standard policy supplies one: once the budget for the current period is exhausted — too many incidents, too much accumulated downtime — new feature releases stop. Engineering effort redirects to stability: fixing the causes of the incidents that burned the budget, hardening the weak points, paying down the risk that got taken on. Only once reliability has recovered enough to replenish some budget does the team resume normal release velocity.

This is the mechanism that makes SLOs more than a reporting exercise — they become a release gate with teeth. A team that's blowing through its error budget every month isn't just failing a dashboard metric; it's supposed to visibly slow its shipping pace until the system is stable again, which is a forcing function that pure good intentions rarely provide on their own. Conversely, a team that's comfortably under budget has concrete, defensible license to move fast and take on the kind of risk — a bold migration, an aggressive experiment — that a looser, feelings-based reliability culture would have talked them out of.
