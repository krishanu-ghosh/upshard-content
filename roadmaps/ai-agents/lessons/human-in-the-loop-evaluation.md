---
title: Human-in-the-loop evaluation
description: Checkpoints where a human approves, edits, or answers a question before a risky agent action proceeds, instead of letting it act fully alone.
roadmap: ai-agents
node: human-in-the-loop-evaluation
group: evaluation-safety-and-deployment
video:
  provider: youtube
  id: "-a2oTS9J67U"
  start: 0
  end: null
creator:
  name: AI Engineer
  channel_url: https://www.youtube.com/@aiDotEngineer
duration_seconds: 900
level: intermediate
prerequisites: [evaluating-agent-performance]
takeaways:
  - Human-in-the-loop design inserts explicit checkpoints where a person approves, edits, or answers a question from the agent before it takes a risky or irreversible action.
  - Staged autonomy — starting an agent in a supervised or shadow mode and graduating it to autonomous operation only for lower-risk actions — lets trust in the agent grow from evidence instead of from assumption.
  - Naming a specific decision owner for each checkpoint, rather than leaving approval to "a human," is what makes the checkpoint actually get exercised instead of silently rubber-stamped.
  - The override rate — how often a human corrects or rejects what the agent proposed — is a concrete, trackable signal of whether the agent is actually ready for more autonomy.
check_questions:
  - q: What is staged autonomy, and why start an agent there instead of giving it full autonomy immediately?
    a: Staged autonomy means running an agent in a supervised or shadow mode first — where it proposes actions a human reviews before they execute — and only letting it act autonomously on lower-risk actions once its proposals have proven reliable; starting fully autonomous skips the evidence-gathering stage where you'd actually discover whether the agent is trustworthy before it can cause damage.
  - q: Why does a high override rate matter even if the agent's overall task success rate looks fine?
    a: The override rate counts how often a human had to correct or reject what the agent proposed before it executed, which measures something a raw success rate doesn't — the agent might only be succeeding because a human is catching and fixing its mistakes before they happen, and a high override rate reveals that dependency even when the end-to-end outcomes look clean.
deeper:
  - label: "Ambient Agents and the New Agent Inbox ft. Harrison Chase — Sequoia Capital"
    url: https://www.youtube.com/watch?v=egSh4TxS5go
related_formulas: []
---

## Why some actions shouldn't be fully autonomous

Everything covered so far in this group assumes you can measure an agent's
quality after the fact — judge its trajectory, trace its steps, compute a
success rate. Human-in-the-loop design is about a different problem: some
actions are expensive or impossible to undo, so you don't want to wait for a
post-hoc evaluation to catch a mistake — you want a person to see the proposed
action before it happens. Sending an email to a customer, issuing a refund,
deleting a database record, or committing code to production are all cases
where "the agent was wrong 2% of the time" is a fine aggregate statistic and a
terrible thing to discover after the fact for the one case that mattered.

## The checkpoint pattern

A human-in-the-loop checkpoint is a point in the agent's flow where, instead
of executing an action directly, the agent pauses and presents the proposed
action — what it's about to do and why — to a person, who can approve it
unchanged, edit it, reject it, or answer a clarifying question the agent
couldn't resolve on its own. The agent resumes only after that response comes
back. This is different from just logging an action after it happened:
logging lets you discover a mistake, a checkpoint prevents it from happening
in the first place. The tradeoff is latency and human attention — every
checkpoint is a point where the agent stops being faster than a human doing
the task directly, so checkpoints need to be reserved for actions where that
cost is worth paying.

## A worked example: a refund agent

Consider a support agent that can issue refunds. Without a checkpoint, the
agent reads a complaint, decides a refund is warranted, calls
`issue_refund(order_id, amount)`, and the money moves immediately. If the
agent misreads the policy, or hallucinates an amount, the refund has already
happened — recoverable in principle, but now it requires a human to notice the
error, investigate, and claw the money back, which costs far more attention
than a checkpoint would have.

With a checkpoint, the agent instead produces a proposal: "Order #8832,
customer reports item arrived damaged, policy supports a refund, proposed
amount $42.00 — approve?" A support lead reviews this in seconds, most of the
time approving as-is. The value isn't that the lead rechecks every detail from
scratch — it's that the one time the agent proposes refunding the full order
total instead of a single damaged item, or misreads which policy applies, the
lead catches it before the money moves rather than after.

## Staged autonomy: earning the right to skip the checkpoint

Running every action through a human checkpoint forever defeats the purpose
of building an agent at all. Staged autonomy is the practice of starting an
agent in a fully supervised mode — every proposed action reviewed before it
executes, sometimes called shadow mode when the agent's proposals are
reviewed but a legacy manual process is still what actually executes — and
only removing checkpoints for specific, lower-risk action types once the
agent's proposals in that category have proven reliable over enough volume.
The refund agent above might start with every refund checkpointed; after a few
thousand approvals with a very low override rate on refunds under $50, that
specific category might graduate to autonomous execution, while refunds over
$500 or refunds tied to a complaint flagged as a potential policy violation
stay checkpointed indefinitely. Autonomy is granted action-by-action and
earned with evidence, not switched on for the whole agent at once.

## Decision owners and the override rate

Two practical details determine whether this pattern actually works. First,
each checkpoint needs a named decision owner — a specific person or role
responsible for reviewing it — rather than a vague "a human reviews this."
An unowned checkpoint tends to get approved reflexively by whoever happens to
see it, which is a checkpoint in name only. Second, the override rate — the
fraction of proposed actions a reviewer edits or rejects rather than approving
as-is — is the metric that tells you whether an agent is actually ready to
graduate to more autonomy. An agent whose refund proposals get overridden 1%
of the time is a plausible candidate for reduced supervision in that category.
One overridden 30% of the time is not ready, regardless of how confident its
own output sounds — and tracking that number over time, broken out by action
type, is what turns "do we trust this agent yet" from a feeling into a number
you can actually watch move.
