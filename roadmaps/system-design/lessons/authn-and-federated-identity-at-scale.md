---
title: "Authentication and Federated Identity at Scale"
description: How a distributed system centralizes authentication with an identity provider and tokens instead of every service checking passwords itself.
roadmap: system-design
node: authn-and-federated-identity-at-scale
group: observability-and-security-at-scale
video:
  provider: youtube
  id: "Vt3rJzITRw4"
  start: 0
  end: null
creator:
  name: Ubisecure
  channel_url: https://www.youtube.com/@Ubisecure
duration_seconds: 1231
level: intermediate
prerequisites:
  - reverse-proxies-and-api-gateways
takeaways:
  - Letting every microservice check passwords and manage credentials itself duplicates sensitive logic dozens of times and multiplies the places a breach can happen.
  - An identity provider (IdP) centralizes authentication once, and every other service trusts its verdict instead of re-implementing login.
  - A token — commonly a JWT — is a signed, self-contained proof of identity that a downstream service can verify without calling back to the identity provider for every request.
  - Federated identity lets a user authenticate with one identity source and be trusted across multiple independent systems, which is the mechanism behind "Sign in with Google" and enterprise single sign-on alike.
  - SAML and OIDC are the two dominant conceptual protocols for this — SAML is XML-based and common in older enterprise single sign-on, OIDC is JSON/JWT-based and built on top of OAuth 2.0.
  - Centralizing authentication also centralizes the blast radius of getting it wrong, which is why the identity provider itself is usually the most heavily hardened and monitored component in the system.
check_questions:
  - q: Why does checking passwords independently in every microservice become a liability as a system grows?
    a: Each service that handles raw credentials is a separate place where credential-handling logic can have a bug, a separate place that needs its own security review, and a separate attack surface, so duplicating that logic across dozens of services multiplies the system's total exposure instead of just its code.
  - q: What problem does a signed token like a JWT solve that a server-side session lookup doesn't?
    a: A signed token carries its own proof of validity inside it, so any service that holds the identity provider's public key can verify the token locally and instantly, without a network round trip back to a central session store for every single request.
  - q: What's the conceptual difference between SAML and OIDC?
    a: Both let a user authenticate once with an identity provider and be trusted by other applications, but SAML exchanges XML-based assertions and is more associated with established enterprise single sign-on, while OIDC is built on top of OAuth 2.0, exchanges JSON Web Tokens, and is the more common choice for newer, especially consumer-facing, systems.
deeper:
  - label: "SSO vs SAML vs OIDC vs JWT: Complete Auth Guide"
    url: "https://www.youtube.com/watch?v=47iHNEuQLTY"
  - label: "Single Sign-On Best Practices: How Organisations can Implement SSO"
    url: "https://www.youtube.com/watch?v=wV9eUle5Vp0"
related_formulas: []
---

## Don't let every service invent its own login

In a monolith, authentication is simple by construction: there's one codebase, one place that checks a password against a database and starts a session, and everything else in the application just trusts that the request arrived with a valid session attached. Split that monolith into dozens of independently deployed services, and the naive approach — have each service independently verify who's calling it — turns into a liability fast. Every service that touches raw credentials is a separate place where that logic can have a bug, a separate thing that needs a security review, a separate system that needs to be kept in sync if a password policy or a token format ever changes. A breach in the least carefully written of those dozens of services is still a breach of the whole system's trust.

The fix mirrors a pattern this roadmap has already covered for other cross-cutting concerns: centralize the thing that's dangerous to duplicate. Instead of every service authenticating users itself, one component — the **identity provider (IdP)** — becomes the single place that actually verifies who someone is: checking a password, validating a second factor, enforcing account lockout policy. Every other service in the system doesn't authenticate users at all; it trusts a verdict the identity provider already rendered.

## Tokens: carrying a verdict without a phone call every time

The naive way to "trust the identity provider's verdict" would be for every service, on every request, to call the identity provider and ask "is this user still logged in, and who are they?" That works, but it turns the identity provider into both a bottleneck and a single point of failure that every single request in the system now depends on synchronously.

The alternative is the **token** — most commonly a **JWT (JSON Web Token)**. When a user authenticates, the identity provider issues a token: a compact, digitally signed bundle of claims (user ID, roles, an expiration time, maybe which organization they belong to). Because it's cryptographically signed with a key only the identity provider holds, any downstream service can verify the token's authenticity using the identity provider's public key — entirely locally, with no network call back to the identity provider required. The token is, in effect, a notarized statement the user carries with them: "the identity provider verified this at 10:04am, here's who they are, and this is valid until 11:04am." Every service that receives a request just checks the signature and the expiration, and it knows exactly as much as if it had called the identity provider directly — without actually making that call.

This is also why token expiration and revocation are real engineering concerns, not footnotes: because a valid, unexpired token is trusted without a round trip, a compromised token stays dangerous until it expires, which is why short expiration windows paired with a separate refresh mechanism are the normal way to bound that exposure.

## Federated identity: trusting someone else's identity provider

**Federated identity** takes the same idea and extends it across organizational boundaries. Instead of your system running its own identity provider from scratch, it trusts an external one — Google, an enterprise's internal Active Directory via an identity broker, a government digital-identity scheme — to vouch for a user's identity, and your system just consumes the resulting token. This is the mechanism behind both "Sign in with Google" on a consumer app and enterprise single sign-on (SSO), where an employee logs into their company's identity provider once and is automatically trusted by dozens of internal tools without a separate login for each.

Two protocols dominate how this trust relationship is actually expressed, and it's worth knowing them at a conceptual level even without memorizing their handshakes. **SAML (Security Assertion Markup Language)** is older, XML-based, and is still the backbone of a great deal of established enterprise SSO — a company's HR system, ticketing system, and internal wiki all trusting the same corporate identity provider, typically wired up by IT rather than by each application's own engineers. **OIDC (OpenID Connect)** is newer, built directly on top of OAuth 2.0, exchanges JSON-based tokens (typically JWTs) instead of XML, and is generally the more natural fit for modern APIs and consumer-facing applications, including most "log in with a third-party account" flows. Both solve the same underlying problem — let one trusted party vouch for identity, let everyone else consume that vouching — and the choice between them in practice usually comes down to which ecosystem a given identity provider and set of client applications already speak.

## The centralization tradeoff, acknowledged

Concentrating authentication into one identity provider removes duplicated risk from every other service, but it doesn't remove risk — it relocates and concentrates it. The identity provider is now the component whose compromise would be catastrophic, which is exactly why real deployments put outsized security effort — hardened infrastructure, aggressive monitoring, short-lived tokens, careful key management for the signing keys themselves — into that one component rather than spreading thin security effort evenly across every service. Centralizing authentication is a trade: you accept a bigger single target in exchange for being able to actually defend it well, instead of having fifty mediocre defenses that only need one failure to be exploited.
