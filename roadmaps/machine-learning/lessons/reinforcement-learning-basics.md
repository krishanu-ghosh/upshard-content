---
title: Reinforcement learning basics
description: An agent learning from an environment's rewards — the agent/environment/state/action/reward loop, explore-vs-exploit, and Q-learning as a concrete example.
roadmap: machine-learning
node: reinforcement-learning-basics
group: deep-learning-frontiers
video:
  provider: youtube
  id: "JgvyzIkgxF0"
  start: 0
  end: null
creator:
  name: Arxiv Insights
  channel_url: https://www.youtube.com/@ArxivInsights
duration_seconds: 987
level: intermediate
prerequisites: [types-of-learning, training-neural-networks]
takeaways:
  - Reinforcement learning has no dataset of correct answers prepared in advance; an agent learns entirely by taking actions in an environment and observing the reward each action leads to.
  - The explore-exploit tradeoff is central — an agent that only ever repeats its current best-known action can never discover a better one, but an agent that only ever explores never benefits from what it has already learned.
  - Q-learning estimates the long-term value of taking a specific action in a specific state, updating that estimate from experience, and ultimately choosing whichever action has the highest estimated value in each situation.
check_questions:
  - q: How does the reinforcement learning setup differ from every other category of learning covered in this roadmap?
    a: There is no dataset of correct input-output pairs prepared in advance. An agent takes actions inside an environment and receives a reward signal afterward, learning over repeated attempts which actions tend to lead toward good outcomes, rather than learning from examples someone already labelled as correct.
  - q: Why can't an agent simply always choose the action it currently believes is best?
    a: Its current belief about which action is best was formed from limited past experience, and a better action it has not yet sufficiently tried might exist. Always exploiting the current best guess forecloses ever discovering that better option, which is exactly why some deliberate exploration has to be built into the agent's behaviour.
deeper:
  - label: "CrashCourse — Reinforcement Learning, Crash Course AI #9"
    url: https://www.youtube.com/watch?v=nIgIv4IfJ6s
  - label: "MATLAB — What Is Reinforcement Learning?"
    url: https://www.youtube.com/watch?v=pc-H4vyg2L4
related_formulas: [discounted-return, q-learning-update, bellman-expectation-equation, action-value-function, state-value-function]
---

## Learning from consequences, not from an answer key

Every form of learning covered elsewhere in this roadmap, from
[linear regression](/roadmaps/machine-learning/linear-regression) through
[generative models](/roadmaps/machine-learning/generative-models), learns
from a dataset prepared in advance — examples already paired with a correct
answer, or at minimum a defined target to reconstruct. Reinforcement learning
has no such dataset at all. Instead, an **agent** takes **actions** inside an
**environment**, observes the resulting **state**, and receives a **reward**
signal reflecting how good or bad that outcome was. Over many repeated
attempts, the agent learns which actions, taken in which situations, tend to
lead toward higher cumulative reward — covered formally in
[discounted return](/formulas/discounted-return), which accounts for the
fact that a reward received later is typically worth somewhat less than the
same reward received sooner.

## Explore versus exploit: the central tension

At every decision point, an agent faces a real tension. **Exploiting** means
choosing whichever action currently looks best based on experience so far —
sensible if that belief is accurate, but it forecloses ever discovering a
genuinely better action the agent has not yet sufficiently tried.
**Exploring** means deliberately trying something other than the current
best guess, purely to gather more information — valuable for discovering
better options, but costly if it means repeatedly passing up a reward the
agent already knows how to get. A reasonable strategy balances both,
typically exploiting the current best-known action most of the time while
still exploring some small fraction of the time, so the agent keeps
improving its knowledge of the environment without entirely sacrificing the
reward it could otherwise be collecting.

## Q-learning: estimating the value of an action

**Q-learning** gives this general idea a concrete algorithm. It maintains an
estimate, called a Q-value, of how good it is to take a specific action in a
specific state — formally, the expected cumulative future reward from taking
that action there and acting well afterward, covered in
[the action-value function](/formulas/action-value-function). Every time the
agent takes an action and observes the resulting reward and new state, it
updates its estimate for that state-action pair slightly, nudging it toward
what was actually just observed — the precise update rule is given in
[the Q-learning update](/formulas/q-learning-update), and the underlying
consistency it is built to satisfy is
[the Bellman expectation equation](/formulas/bellman-expectation-equation).
Over enough repeated experience, these estimates converge toward accurate
values, at which point the agent can act well simply by choosing whichever
action has the highest estimated value in its current state.

## A deliberately brief introduction

This lesson introduces the vocabulary and the core loop that every
reinforcement learning method builds on, using Q-learning as one concrete,
foundational example of how an agent actually turns experience into improved
behaviour. The field extends considerably further than this single lesson
covers — combining reinforcement learning with the neural networks covered
earlier in this roadmap to handle environments with far too many possible
states to track individually is itself an entire area of further study,
touched on only briefly here as the natural next step for anyone who wants to
go beyond this roadmap's scope.
