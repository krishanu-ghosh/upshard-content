---
title: "Zero Trust Architecture"
description: "Why \"never trust, always verify\" replaced the perimeter model, and what actually enforces it: identity, device posture, and micro-segmentation."
roadmap: software-architect
node: zero-trust-architecture
group: security-architecture
video:
  provider: youtube
  id: "yn6CPQ9RioA"
  start: 0
  end: null
creator:
  name: IBM Technology
  channel_url: https://www.youtube.com/@IBMTechnology
duration_seconds: 222
level: intermediate
prerequisites:
  - owasp-top-10-for-architects
takeaways:
  - "The perimeter model assumes everything inside the corporate network is trustworthy and everything outside is not; zero trust assumes neither — every request is verified on its own merits, regardless of where it originates."
  - "\"Never trust, always verify\" is enforced continuously, not once at login — a zero-trust system re-evaluates identity, device posture, and context (location, time, behavior) on every request, not just at the start of a session."
  - "Micro-segmentation breaks a network into small, independently controlled zones so that compromising one service doesn't grant an attacker lateral access to everything else — the architectural opposite of one flat network where anything inside can reach anything else."
  - "Identity federation (SAML/OAuth/OIDC, covered in system-design's authn-and-federated-identity-at-scale) is a building block zero trust relies on, not a replacement for it — proving who is asking is necessary but not sufficient without also checking device health and enforcing least-privilege access."
  - "Zero trust is a model and a set of enforcement points (policy decision points, policy enforcement points), not a single product — a vendor selling \"a zero trust solution\" is selling one piece of an architecture an organization still has to assemble and operate."
check_questions:
  - q: What's the core assumption zero trust rejects that the perimeter model depends on?
    a: The perimeter model trusts a request based on where it comes from — inside the corporate network is safe, outside is not; zero trust rejects that assumption entirely and verifies every request on identity, device posture, and context regardless of network location.
  - q: Why does zero trust verify continuously instead of just at login?
    a: A session that was legitimate at login can become compromised afterward — a stolen session token, a device that gets infected mid-session — so continuous verification re-checks identity and context on an ongoing basis rather than issuing one blanket trust decision that lasts for the whole session.
  - q: How does micro-segmentation limit the damage of a single compromised service?
    a: Without segmentation, one compromised service on a flat network can reach every other service on that network; micro-segmentation puts each service or small group of services in its own enforced zone, so a compromise is contained to what that zone can reach, not the whole network.
deeper:
  - label: "Cybersecurity and Zero Trust"
    url: "https://www.youtube.com/watch?v=FMMWSLIcaME"
related_formulas: []
---

## The model the perimeter replaced

For decades, enterprise network security worked like a castle: a hardened perimeter (firewalls, VPN gateways) separated a trusted inside from an untrusted outside, and once a device or user was inside — on the corporate network, through the VPN — it was broadly trusted to reach most of what was there. This worked reasonably well when "inside" meant a physical office network. It stopped working as employees went remote, applications moved to clouds the company didn't operate, and attackers got good at one specific move: compromise a single device or credential at the edge, then move **laterally** across the trusted interior, which had comparatively little internal friction to stop them.

**Zero trust** rejects the premise. Instead of "trusted inside, untrusted outside," every request is verified on its own merits regardless of where it originates — a request from an employee's laptop on the corporate Wi-Fi gets exactly as much scrutiny as one from a coffee shop. The shorthand for this is **"never trust, always verify."**

## Verification is continuous, not a one-time gate

The perimeter model's trust decision happens once: authenticate at the VPN gateway or the office door, then move relatively freely. Zero trust's trust decision is re-evaluated **continuously** — on every request, or close to it. A policy decision point considers several signals together before granting access to a specific resource:

- **Identity** — who is asking, authenticated with strong multi-factor credentials, not just a password.
- **Device posture** — is the requesting device managed, patched, running expected security software, not jailbroken or showing signs of compromise?
- **Context** — is this request consistent with the user's normal location, time of day, and behavior, or does it look anomalous?

Identity federation protocols — SAML, OAuth, OIDC, covered in `system-design`'s `authn-and-federated-identity-at-scale` — are the mechanism that answers the "who" question, and zero trust depends on them. But identity alone isn't zero trust: a valid, correctly authenticated user on a compromised or non-compliant device is still a risk zero trust is designed to catch, which is why device posture and context sit alongside identity rather than being replaced by it.

## Least privilege and micro-segmentation

Verifying identity and context on every request only helps if what's granted afterward is narrow. Zero trust pairs continuous verification with **least-privilege access**: a verified request gets access to exactly the specific resource it asked for, for as long as it needs it, not standing access to a broad network segment. **Micro-segmentation** is how this gets enforced structurally — instead of one flat network where any internal host can reach any other, the network (or the service mesh, in a cloud-native system) is divided into small zones, each with its own enforced policy about what can talk to what. If an attacker compromises one service, micro-segmentation is what stops that compromise from becoming lateral movement into everything else; without it, the blast radius of a single breach is the whole network, which is precisely the perimeter model's failure mode zero trust exists to close.

## What zero trust is not

It's tempting to read "zero trust" as a product category, because plenty of vendors sell one under that label. It's more accurate to treat it as an **architectural model** implemented through a combination of pieces an organization assembles: identity providers, device-management and posture-checking tools, policy decision and enforcement points, and a segmented network or service mesh that actually carries out the per-request decisions those pieces make. Buying one vendor's "zero trust" product typically covers one piece of this — strong identity, say, or network segmentation — not the whole model. It's also distinct from network-layer defenses like DDoS mitigation (`system-design`'s `ddos-mitigation-and-rate-limiting-for-security`), which protect availability against volumetric attacks; zero trust is about authorization and access decisions for legitimate-looking traffic, a different layer of the same overall security posture.

## Why this is an architectural decision, not a tooling purchase

Adopting zero trust means deciding, system by system, what the smallest useful segmentation boundary is, which signals a policy decision point actually needs to evaluate, and how much friction continuous verification adds to legitimate use versus how much lateral-movement risk it closes off. None of that is answered by a product datasheet — it's a design exercise an architect does once per system, the same way choosing an architectural style or a data model is, and it's exactly the kind of decision this roadmap's `architecture-decision-records-and-tradeoffs` lesson is about recording well.
