---
title: "Secrets and Config Management"
description: Why secrets and per-environment config don't belong in code or images, and how centralized secrets stores and rotation reduce real risk.
roadmap: system-design
node: secrets-and-config-management
group: observability-and-security-at-scale
video:
  provider: youtube
  id: "VYfl-DpZ5wM"
  start: 0
  end: null
creator:
  name: HashiCorp, an IBM Company
  channel_url: https://www.youtube.com/@HashiCorp
duration_seconds: 1012
level: intermediate
prerequisites:
  - authn-and-federated-identity-at-scale
takeaways:
  - A secret (a database password, an API key, a signing key) committed to source control stays exposed forever, even after it's deleted, because it lives on in the repository's history.
  - Baking a secret into a container image means anyone who can pull that image — including from an old, supposedly-retired build — can read the secret out of it.
  - A centralized secrets store, in the style of HashiCorp Vault, holds secrets once and hands them out to services at runtime over an authenticated channel, instead of each service shipping with its own copy.
  - Config (feature flags, per-environment URLs, timeout values) is different from a secret in sensitivity but shares the same underlying problem — it shouldn't be hardcoded into a build that then needs to be rebuilt just to change a value.
  - Rotation — periodically replacing a secret with a new one and invalidating the old one — bounds how long a leaked credential stays useful to whoever stole it.
  - Dynamic secrets, issued on demand with a short lifespan and automatically revoked, go a step further than rotation by minimizing how long any given credential exists at all.
check_questions:
  - q: Why is deleting a secret from the latest commit not enough to make it safe?
    a: Source control keeps history, so the secret is still readable in every earlier commit that contained it; removing it from the current file doesn't remove it from the repository, and anyone with read access to the history can still retrieve it.
  - q: What does a centralized secrets store change about how a service gets its database password?
    a: Instead of the password being baked into the service's config file or image at build time, the service authenticates to the secrets store at startup or runtime and requests the password over a secured channel, so the secret lives in one well-guarded place instead of being copied into every service that needs it.
  - q: How does rotating a secret limit the damage from a leak that already happened?
    a: If a credential leaks and is still valid indefinitely, whoever has it can use it indefinitely; rotating it on a schedule means even an already-leaked secret stops working once the next rotation happens, bounding the attacker's window instead of leaving it open forever.
related_formulas: []
---

## Secrets in source control and container images are a slow-motion leak

A **secret** is any credential that grants access: a database password, a third-party API key, a private signing key, an encryption key. The most common mistake with secrets isn't exotic — it's simply putting them where code already lives. A password dropped into a config file that gets committed to source control, or an API key hardcoded into a script "just for now," feels harmless in the moment, especially on a private repository. It isn't. Source control is designed to remember everything, which is exactly the wrong property for something you later want to be able to take back. Deleting the secret from the latest commit doesn't delete it from history; it's still sitting in every earlier commit, retrievable by anyone with read access to that history, including anyone who gets that access well after the fact — a contractor whose access should have been revoked, a fork someone made two years ago, a breach of the hosting provider itself.

Baking secrets into a **container image** has the same failure mode with an extra twist: images get pushed to registries, pulled onto build servers, cached on developer laptops, and sometimes kept around as "known good" rollback targets long after the service has moved on. A secret embedded in an image layer is retrievable by anyone who can pull that image, and "anyone who can pull that image" is a much larger and longer-lived set of people than "anyone who currently has access to the running service." Both mistakes share the same root cause: treating a secret as if it were just another piece of configuration that ships with the build, rather than something that needs to be handled completely separately from code.

## Centralizing secrets instead of copying them everywhere

The fix that's become standard practice is a **centralized secrets store** — HashiCorp Vault is the name most associated with the pattern, though the idea generalizes to any equivalent service. Rather than every service carrying its own copy of every secret it needs, baked in at build time, secrets live in exactly one place, and services retrieve them at runtime over an authenticated, encrypted channel. A service starting up authenticates itself to the secrets store — often using the same kind of identity mechanism covered in the previous lesson, proving it's actually the `payments-service` and not an impostor — and only then is it handed the database password or API key it needs, held in memory, never written to disk or committed anywhere.

This buys several things at once that scattering secrets everywhere can't. There's exactly one place to audit who accessed which secret and when, instead of guessing which of fifty repositories and image registries might contain a copy. There's exactly one place to revoke access if a service or a person is compromised. And critically, rotating a secret becomes a change made in one place that every consuming service picks up automatically on its next request, instead of a change that requires hunting down and rebuilding every service that happened to have a hardcoded copy.

## Config isn't a secret, but it has the same disease

**Configuration** — a feature flag, a per-environment service URL, a timeout value, a retry count — usually isn't sensitive the way a password is, but it suffers from a closely related problem: hardcoding it into a build means changing it requires rebuilding and redeploying, even for a one-line value change, and it means the same build artifact can't be reused as-is across a staging environment and a production environment that need slightly different values. A **config service** externalizes these values the same way a secrets store externalizes credentials: the application reads its configuration at startup (or, for things like feature flags, continuously) from a central source, and the exact same build artifact can run correctly in any environment, or have a value tweaked live, without anyone touching code or waiting on a deploy pipeline. The underlying lesson is the same one driving the whole observability-and-security group of this roadmap: anything that varies independently of your application's logic — its telemetry, its identity, its secrets, its configuration — works better pulled out into its own well-defined system than smeared throughout application code.

## Rotation: bounding how long a leak matters

Even a perfectly centralized secret can still leak — through a misconfigured access policy, a compromised service, a careless log line that captured it by accident. **Rotation** is the practice of periodically replacing a secret with a new value and invalidating the old one, on a schedule, whether or not you have any specific reason to suspect a leak. The value of rotation isn't preventing leaks; it's bounding how long any one leaked credential stays useful. A password that never rotates is a liability forever once it's exposed; a password that rotates every 30 days limits an undetected leak's usefulness to, at most, 30 days.

The more advanced version of this idea is the **dynamic secret**: instead of a long-lived credential that gets rotated on a calendar, the secrets store issues a brand-new, short-lived credential on demand every time a service needs one — a database password valid for exactly one hour, say — and automatically revokes it afterward. Rather than minimizing how often a long-lived secret changes, dynamic secrets minimize how long any single secret exists at all, which shrinks the window an attacker would need to exploit a leak down to something close to zero.
