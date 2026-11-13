---
title: Data poisoning and backdoor triggers
description: Poisoning corrupts a model's training signal to skew its behavior, and backdoor triggers hide that corruption behind normal-looking accuracy.
roadmap: ai-red-teaming
node: data-poisoning
group: model-attacks
video:
  provider: youtube
  id: "ahC4KPd9lSY"
  start: 0
  end: null
creator:
  name: IEEE Symposium on Security and Privacy
  channel_url: https://www.youtube.com/@ieeessp
duration_seconds: 1020
level: advanced
prerequisites: [threat-modeling-for-ai-systems]
takeaways:
  - Data poisoning corrupts a model's training data rather than its weights or code directly, so a small, attacker-chosen fraction of bad examples can skew what the model later generalizes to believe.
  - Fine-tuning datasets, RAG document stores, and user-feedback signal are all realistic poisoning surfaces an external attacker can often influence, even when a foundation model's pretraining data is completely out of reach.
  - A backdoor is a poisoning attack engineered to behave normally on every input except one attacker-chosen trigger, which is exactly why normal accuracy evaluation fails to catch it.
  - Detecting poisoning requires auditing training data provenance and probing for conditional misbehavior, not just measuring aggregate accuracy on a held-out set.
check_questions:
  - q: Why can a poisoned model still pass a normal accuracy evaluation?
    a: Because poisoning, especially a backdoor, is engineered to misbehave only on a narrow, attacker-chosen trigger that almost never appears in a held-out evaluation set, so average accuracy looks unaffected.
  - q: What are three realistic surfaces an outside attacker can poison today, even without access to a foundation model's pretraining data?
    a: Fine-tuning datasets, documents in a RAG retrieval corpus, and user feedback or preference signal used for ongoing tuning are all surfaces an external party can often influence.
  - q: What makes a backdoor different from a general drop in model accuracy caused by poisoning?
    a: A backdoor leaves the model behaving completely normally on every input except a specific, attacker-chosen trigger, rather than degrading performance broadly, which is what lets it hide from routine evaluation.
deeper:
  - label: "USENIX Enigma 2020 - What Does It Mean for Machine Learning to Be Trustworthy? — Nicolas Papernot"
    url: https://www.youtube.com/watch?v=TJOLm0FM3PA
related_formulas: []
---

## Corrupting the signal, not the system

Every attack covered earlier in this roadmap's model-attacks group treats
the trained model as a fixed, finished artifact and tries to extract
information from it after the fact. Data poisoning attacks the step before
that: the training process itself. An attacker who can influence some
fraction of the data a model learns from can shift what the model
generalizes to believe, without ever touching its code, its serving
infrastructure, or its weights directly — the corruption is laundered
through ordinary gradient descent. Matthew Jagielski's research on poisoning
attacks and countermeasures demonstrated this concretely on classic
supervised learning: inject a relatively small fraction of mislabeled or
adversarially crafted examples into a training set, and a model trained on
the resulting mixture can be pushed toward systematically wrong predictions
on inputs the attacker cares about, while performing close to normally
everywhere else.

That "close to normally everywhere else" clause is what makes poisoning a
genuinely different threat model from the ones before it in this roadmap. A
corrupted training process doesn't produce an obviously broken model; it
produces a model that passes a normal accuracy benchmark while harboring a
specific, attacker-chosen failure the benchmark was never designed to find.

## Where an attacker can actually reach the data today

Poisoning a frontier model's pretraining corpus is, for almost any realistic
attacker, out of reach — that data is enormous, curated, and controlled by
the lab training the model. But that was never the only surface, and for the
systems a red team is actually asked to assess, it's rarely even the
relevant one. Three surfaces are realistically reachable by an external
party:

- **Fine-tuning data.** A model customized on a smaller, domain-specific
  dataset inherits whatever bias that dataset contains, and that dataset is
  often smaller, less curated, and sourced from less-vetted places than a
  foundation model's pretraining run — exactly the conditions that make a
  small poisoned fraction go further.
- **RAG document stores.** A retrieval-augmented system treats whatever it
  retrieves as trustworthy context. If the corpus it retrieves from includes
  anything an outside party can edit — a public wiki, a forum, a
  documentation site the pipeline periodically re-indexes — an attacker
  doesn't need to touch the model at all. They edit the page; the pipeline
  retrieves the attacker's version later and feeds it to the model as if it
  were trusted, with no poisoning of a training run in the traditional sense.
- **User feedback and RLHF-style signal.** A model tuned on thumbs-up or
  thumbs-down ratings, or on other user-supplied preference signal, is being
  trained — slowly, incrementally — on whatever the userbase submits. An
  attacker willing to submit many coordinated ratings is poisoning a live
  training loop, not a one-time dataset.

## Backdoors: a trigger, not a general skew

The most operationally dangerous form of poisoning isn't a general accuracy
degradation — that tends to get caught by routine evaluation. It's a
backdoor: the attacker engineers the poisoned examples so the model learns
an association between one specific, unusual trigger (a particular phrase,
an unusual image patch, a rare token sequence) and an attacker-chosen
output, while behaving completely normally on every input that doesn't
contain the trigger. A backdoored classifier scores exactly like the clean
version on the entire evaluation set, because the evaluation set almost
certainly contains zero instances of the trigger. The behavior only appears
for the attacker, on demand, when they supply the trigger — which also makes
a backdoor a durable, patient attack: it can sit inert through months of
normal operation and evaluation cycles, activated only when the attacker
chooses to use it.

## Why standard testing misses this

Ordinary model evaluation measures aggregate performance against a held-out
set drawn from the same distribution as the training data. A backdoor is
specifically engineered to be invisible to exactly that measurement, because
the trigger condition is, by construction, outside the distribution the
evaluation set samples from. Catching poisoning requires different tooling:
auditing the training data's provenance before it's trusted, looking for
anomalous clusters or outlier gradients in the training set, and testing the
finished model against inputs deliberately chosen to probe for exactly this
kind of narrow, conditional misbehavior rather than only measuring average
accuracy — a theme that reappears when this roadmap gets to automated and
adversarial testing methodology.
