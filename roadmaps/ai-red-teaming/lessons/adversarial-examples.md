---
title: Adversarial examples and black-box attacks
description: Adversarial examples fool a model with inputs that look unchanged to a human, and black-box attacks craft them using only output labels.
roadmap: ai-red-teaming
node: adversarial-examples
group: model-attacks
video:
  provider: youtube
  id: "vkCifg2rp34"
  start: 0
  end: null
creator:
  name: IEEE Symposium on Security and Privacy
  channel_url: https://www.youtube.com/@ieeessp
duration_seconds: 1020
level: advanced
prerequisites: [data-poisoning]
takeaways:
  - An adversarial example is an input with a small, deliberately computed perturbation that looks unchanged to a human but flips a model's prediction, because the perturbation pushes the input across a decision boundary that sits closer than intuition suggests.
  - HopSkipJumpAttack shows that crafting adversarial examples doesn't require gradient access or confidence scores; querying only a model's final predicted label is enough to estimate a boundary-crossing direction.
  - Adversarial suffixes against language models are the text analogue of image perturbations — short, often meaningless token sequences discovered by automated search that reliably break a model's alignment.
  - Most published defenses against adversarial examples have been broken by a slightly stronger, adaptive attacker, which is why robustness claims should always specify what attack they were actually tested against.
check_questions:
  - q: Why can an adversarial image example fool a model even though it looks identical to the original to a human?
    a: Because the model's decision boundary passes much closer to the original input than human visual judgment would suggest, and a small, deliberately computed perturbation is enough to push the input across that nearby boundary.
  - q: What makes HopSkipJumpAttack a realistic threat against a production API?
    a: It crafts adversarial examples using only the model's final predicted label, with no gradient access and no confidence scores, which is exactly the limited information a locked-down production API typically exposes.
  - q: What is an adversarial suffix, and how is it typically found?
    a: It is a short, often meaningless string of characters or tokens appended to a prompt that reliably breaks a language model's alignment, and it is usually discovered through automated search rather than written by a human.
  - q: What did Carlini's evaluation of adversarial defenses find, and why does it matter?
    a: It found that most published defenses are broken once tested against an attacker adapted specifically to that defense's mechanism, which is why robustness should be read as currently resistant to known attacks rather than solved.
deeper:
  - label: "USENIX Security '19 - Lessons Learned from Evaluating the Robustness of Defenses to Adversarial Examples — Nicholas Carlini"
    url: https://www.youtube.com/watch?v=ZncTqqkFipE
related_formulas: []
---

## An invisible nudge that flips the answer

The canonical demonstration of an adversarial example is a pair of images
that look identical to a human and are classified completely differently by
a neural network: a photo, correctly labeled, and the same photo with a
carefully computed perturbation added to every pixel — a change far too
small to perceive — that the classifier now confidently labels as something
else entirely, often with higher confidence than its original correct
answer. This isn't the model simply being wrong in the way an under-trained
model is wrong. The clean image is classified correctly, and an input that
is, by any reasonable human judgment, the same image is classified with
total confidence as something else. The perturbation isn't noise; it's a
direction in the model's input space, found deliberately, that the model's
decision boundary happens to be unusually close to at that particular point —
pushing the input just across that nearby boundary produces the flip.

What makes this a security problem rather than just a curiosity about neural
network geometry is that these perturbations can be found systematically
rather than by luck, and a model's reliance on surface statistical patterns
rather than anything resembling human visual concepts means that boundary is
often much closer than intuition suggests. A model can have excellent,
state-of-the-art accuracy on normal data and still have decision boundaries
that pass alarmingly close to nearly every correctly classified input.

## Attacking a model you can't see inside

The earliest adversarial example attacks assumed white-box access: the
attacker has the model's weights and can compute gradients directly,
letting them calculate exactly which direction in input space moves a point
across the nearest boundary fastest. That's a strong assumption a red
teamer rarely gets against a real deployed system, which usually exposes
only an API. HopSkipJumpAttack, demonstrated by Chen, Jordan, and Wagner,
showed that gradient access isn't actually necessary — a much weaker, far
more realistic query model is enough. Querying only the model's final
predicted label, with no confidence scores and no gradients at all, the
attack starts from a point already on the wrong side of the decision
boundary and iteratively walks that point back toward the original input
while repeatedly querying the model to stay just on the "fooled" side of the
boundary, using the geometry of many such boundary-adjacent queries to
estimate which direction to move next. The result converges toward a small,
often visually imperceptible perturbation using only the information a
completely locked-down production API would realistically expose. The
practical implication for a red teamer is important: a model served only
through a plain classification API, with no probability scores and no
explanation output, is not thereby safe from this entire attack family — it
only raises the number of queries required, which is a cost, not a wall.

## From pixels to tokens: adversarial examples in text

The same underlying phenomenon — a small, carefully chosen perturbation that
exploits a model's decision boundary rather than its intended behavior —
shows up against language models as the adversarial suffix: a short, often
semantically meaningless string of characters or tokens, appended to an
otherwise ordinary prompt, that reliably causes an aligned model to comply
with a request it would normally refuse outright. Like an image
perturbation, these suffixes are typically not written by a human reasoning
about language; they're discovered by automated search procedures that
optimize the suffix against the model's own internal signal where available,
or its outputs where it isn't, the same way a pixel perturbation is
optimized against a classifier. The suffix doesn't "persuade" the model in
any sense a human jailbreak attempt does — it's closer to finding the same
nearby, unintended boundary-crossing that HopSkipJumpAttack finds in image
space, just in the discrete space of token sequences.

## An arms race, not a solved problem

A large body of published defenses has claimed to make models robust
against adversarial examples — gradient masking, input transformations,
detection networks sitting in front of the classifier. Nicholas Carlini's
systematic evaluation of these defenses found that the overwhelming majority
fail once evaluated against an adaptive attacker who specifically targets
the defense's own mechanism rather than being tested only against the
original, undefended attack. A defense that blocks the exact perturbation
style it was designed around routinely falls to a slightly adjusted attack
built with the defense itself in mind. That finding is the reason this
roadmap treats adversarial robustness later, in its defense group, as an
ongoing arms race rather than a checkbox: "robust" in this literature means
"not yet broken by a known attack," never "provably unbreakable," and a red
team evaluating a robustness claim should always ask what attacker the claim
was actually tested against.
