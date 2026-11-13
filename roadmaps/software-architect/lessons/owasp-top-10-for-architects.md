---
title: "The OWASP Top 10 for Architects"
description: "The ten most common web application security risks, read as design decisions an architect makes up front rather than bugs a pentest finds later."
roadmap: software-architect
node: owasp-top-10-for-architects
group: security-architecture
video:
  provider: youtube
  id: "9HtxKEz59Gk"
  start: 0
  end: null
creator:
  name: Cyber World YT
  channel_url: https://www.youtube.com/@CyberWorldYTOfficial
duration_seconds: 276
level: intermediate
prerequisites:
  - public-key-infrastructure
takeaways:
  - "The OWASP Top 10 is a ranked list of the most common and impactful web application security risks, maintained by the OWASP Foundation and revised roughly every few years from real incident and survey data."
  - "This is the general web-application list — a different, unrelated document from the OWASP Top 10 for LLM Applications that ai-red-teaming's own lesson covers; the two share a name and a format, not a topic."
  - "Broken access control and cryptographic failures consistently rank at or near the top because they are architectural — which layer enforces authorization, and whether sensitive data is encrypted at rest and in transit — not line-level coding mistakes."
  - "Security misconfiguration and vulnerable/outdated components both point at the same architectural failure: nobody owns keeping defaults hardened and dependencies patched as an ongoing process, so the risk accumulates silently until something exploits it."
  - "Treating the Top 10 as an architect's checklist means asking \"what did we design in to prevent this class of risk structurally\" at design time, rather than waiting for a penetration test to surface an instance of it after the system is built."
check_questions:
  - q: Why do broken access control and cryptographic failures rank higher than most individual coding mistakes?
    a: Both are architectural decisions — which layer enforces who can access what, and whether sensitive data is protected by default — so getting them wrong doesn't produce one bug, it produces a systemic gap that affects every feature built on top of that layer.
  - q: How does the general OWASP Top 10 differ from the OWASP Top 10 for LLM Applications?
    a: They're separate documents from the same foundation covering different domains — this lesson's list is about classic web-application risks like injection and broken access control, while the LLM-specific list (covered elsewhere on this site, in ai-red-teaming) addresses risks unique to applications built around large language models, like prompt injection and training data poisoning.
  - q: What's the architectural difference between finding a Top 10 risk in a pentest versus designing against it up front?
    a: A pentest finds one instance of a risk after the system already exists, which gets patched in isolation; designing against the risk class up front — centralizing authorization, encrypting by default, patching dependencies on a schedule — closes the whole category everywhere it could occur, including places nobody has tested yet.
deeper: []
related_formulas: []
---

## A ranked list, not a checklist of bugs

The **OWASP Top 10** is maintained by the OWASP Foundation and periodically rebuilt from a combination of large-scale vulnerability data contributed by security vendors and a community survey of practitioners — it's an attempt to rank which risk *categories* cause the most real-world damage, not an exhaustive list of every possible vulnerability. The most recent major revisions kept the same core shape even as specific entries moved: broken access control, cryptographic failures, and injection remain fixtures near the top because they recur across almost every kind of application, while newer categories like insecure design and software/data integrity failures have been added as the field's understanding of what actually causes breaches has matured.

One thing worth saying plainly: this is the **general web-application** Top 10. OWASP publishes a separate, unrelated list — the OWASP Top 10 for LLM Applications — covering risks specific to systems built around large language models, like prompt injection and training-data poisoning; `ai-red-teaming`'s own lesson on this site covers that one. The two lists share a name and a ten-item format because OWASP reused a format that works, not because they cover the same ground.

## The categories that are architectural, not line-level

A few entries on the list are really about where a decision gets made in the system, which is exactly the kind of thing an architect — not just an individual developer — controls:

- **Broken access control** means a user can act outside their intended permissions: editing another user's data by guessing an ID in a URL, or reaching an admin endpoint with no further check. This is almost always a sign that authorization logic is scattered across individual handlers instead of centralized in one place the architecture enforces consistently — a service boundary, a middleware layer, a policy engine — rather than re-implemented (and inevitably, somewhere, forgotten) per endpoint.
- **Cryptographic failures** cover sensitive data transmitted or stored without adequate protection — plaintext passwords, unencrypted data at rest, weak or outdated algorithms still in use. The fix isn't "use better crypto" so much as "decide once, at the architecture level, what's encrypted by default and where keys live," so no individual feature team has to remember to opt in.
- **Security misconfiguration** and **vulnerable and outdated components** are really the same underlying failure from two directions: nobody owns the ongoing process of keeping default configurations hardened and dependencies patched. A secure design on day one degrades silently if patching isn't a scheduled, owned responsibility rather than a one-time setup task.

## The categories that are about trusting input and identity

**Injection** (SQL, command, and otherwise) happens when untrusted input is concatenated into a command or query instead of passed as a separate, parameterized value — an old risk, but still common wherever a codebase mixes user data with executable syntax by hand instead of through a library that keeps them apart. **Identification and authentication failures** cover weak session handling, credential stuffing, and missing multi-factor authentication — the mechanisms that establish *who* is making a request, which every access-control decision downstream depends on. **Server-side request forgery (SSRF)** is a more modern entry: a server that fetches a URL on a user's behalf (an image proxy, a webhook validator) can be tricked into fetching an internal address instead — `http://169.254.169.254/` on most clouds, the metadata endpoint — turning a convenience feature into a way to reach infrastructure the public internet was never supposed to touch.

**Insecure design** and **software and data integrity failures** round out the newer end of the list: insecure design flags that some vulnerabilities exist because the *design itself* never considered the threat (no amount of careful coding fixes a missing rate limit on a password-reset flow that was never designed to need one), and integrity failures cover trusting code or data — a CI/CD pipeline, a software update, a deserialized object — without verifying it hasn't been tampered with.

## Reading the list as an architect

The useful move is to read each category as a question asked at design time, not a bug found at test time: *where* is access control enforced, and is it one place or many? *What* is encrypted by default, and who decided? *Is* there an owned process for patching, or does hardening only happen once, at launch? A penetration test will eventually find an instance of whatever the architecture failed to prevent structurally — the Top 10's real value to an architect is as a list of categories to design against before that test ever runs, not a list of findings to fix after.
