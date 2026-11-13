---
title: "JWT, API Keys, and Session Auth"
description: Four ways to authenticate a request — JWT structure and the mistakes that break it, API keys, Basic Auth, session cookies — and which fits which consumer.
roadmap: api-design
node: jwt-api-keys-and-session-auth
group: authentication-and-authorization
video:
  provider: youtube
  id: "I747kI_y9eQ"
  start: 0
  end: null
creator:
  name: LearnThatStack
  channel_url: https://www.youtube.com/@LearnThatStack
duration_seconds: 801
level: intermediate
prerequisites:
  - oauth2-and-oidc-in-practice
takeaways:
  - "A JWT is three base64url segments — header, payload, signature — joined by dots. The header and payload are only encoded, not encrypted, so anyone can read them; the signature guarantees integrity, not confidentiality. Never put secrets in a JWT payload."
  - "Validating a JWT means checking the signature against the issuer's actual key, then separately checking `exp` (not expired), `nbf` (not used too early), and `aud` (issued for this API, not some other one) — skipping any one of these turns a signed token into an unsigned bearer token for whoever crafts the payload."
  - "The classic JWT vulnerability is a library that trusts the `alg` field the token itself claims: an attacker sends `alg: none` or swaps RS256 for HS256 using the issuer's public key as the HMAC secret, and a naive verifier accepts it. Always pin the expected algorithm server-side; never read it from the token."
  - "API keys carry no user context — they identify the calling application or account, not a person — which is exactly right for server-to-server calls and exactly wrong for anything that needs to know which human is acting."
  - "Basic Auth sends a base64-encoded (not encrypted) username and password on every single request, so it's only acceptable over TLS and mostly only sane for internal, trusted-network, or legacy-system calls today — it has no token, no expiry, no scope."
  - "Session cookies keep state on the server and send only an opaque session ID to the client, which makes revocation instant (delete the server-side session) at the cost of needing shared session storage across every instance of your API — the opposite tradeoff from a stateless JWT, which revokes hard but scales without a shared store."
check_questions:
  - q: Why is a JWT with a valid-looking signature still potentially a forgery if your verification code reads the signing algorithm from the token's own header?
    a: An attacker can set `alg` to `none` (some libraries treat that as "no signature required" and accept it) or switch an RS256 token to HS256 and sign it with the issuer's public RSA key treated as an HMAC secret — since RSA public keys are, by design, public, the attacker has everything needed to forge a token an algorithm-agile verifier will accept. The fix is to hardcode the expected algorithm in the verification call rather than trusting the token to say what it is.
  - q: Why does checking a JWT's `exp` claim alone not catch every validation mistake, and what do `nbf` and `aud` add?
    a: The `exp` claim only stops a token from being used after it expires; `nbf` (not-before) stops a token from being used before it's meant to become valid, which matters for tokens issued for future activation; `aud` (audience) stops a token correctly issued and still live, but meant for a *different* API, from being accepted by yours — without an audience check, any valid token from the same issuer works everywhere that issuer's tokens are trusted, regardless of what it was actually scoped for.
  - q: A third-party integration partner needs to call your API from their backend, with no end user involved and no need for fine-grained per-user permissions. What's the simplest correct choice among the four, and why not a session cookie?
    a: An API key — it identifies the partner's application, requires no login flow, and is trivial to issue and revoke per integration. A session cookie assumes a browser-based client that can store and resend a cookie across requests to the same origin, which doesn't describe a server-to-server integration at all.
  - q: Why does a stateless JWT make instant revocation harder than a session cookie does, and what's the usual workaround?
    a: A session cookie's session ID is just a lookup key — the server deletes the row and the session is dead immediately. A JWT is self-contained and valid until `exp` on its own, with no server lookup involved, so revoking one early means either keeping a server-side denylist (which reintroduces the shared-state problem JWTs were meant to avoid) or living with short expiries and relying on refresh-token rotation to limit the damage window instead.
deeper:
  - label: "Session Vs JWT: The Differences You May Not Know!"
    url: "https://www.youtube.com/watch?v=fyTxwIa-1U0"
related_formulas: []
---

## Four answers to "how does this request prove who's calling"

`oauth2-and-oidc-in-practice` covers how a client *gets* a token in the first place. This lesson is about what happens on your API's side once a request shows up — because OAuth2 is overkill for a large share of the APIs you'll actually build, and picking the right authentication mechanism for the consumer in front of you matters more than defaulting to whichever one is trendiest. There are really four shapes in common use, and they trade off along the same two axes every time: does the credential carry user context, and does validating it require a server-side lookup.

## JWT: structure, claims, and where it actually breaks

A JSON Web Token is three dot-separated segments, each base64url-encoded: a **header** naming the signing algorithm and token type, a **payload** of claims (standard ones like `sub`, `exp`, `iat`, `aud`, plus whatever custom claims your API needs), and a **signature** computed over the header and payload with the issuer's key. The header and payload are encoded, not encrypted — paste any JWT into a decoder and you'll read every claim in plain text. That's by design: a JWT's job is integrity (you can trust it wasn't tampered with) and authenticity (you can trust who issued it), not secrecy. If a claim is sensitive, it doesn't belong in a JWT payload at all.

Validating a JWT correctly means four checks, not one:

1. **Signature.** Recompute it with the issuer's actual public key (RS256) or shared secret (HS256) and compare. This is the step that proves the token wasn't forged or altered.
2. **`exp`** — reject anything past its expiry.
3. **`nbf`** — reject anything used before its "not valid until" time, for tokens issued ahead of their activation.
4. **`aud`** — reject anything not addressed to your API specifically. A token that's otherwise perfectly valid but was issued for a *different* service should bounce here.

The mistake that keeps showing up in real incidents is algorithm confusion: a verification library that reads the `alg` field out of the token's own header and uses whatever it says. An attacker who wants to forge a token sets `alg` to `none` — some libraries interpreted a missing algorithm as "skip verification" — or, if your API expects RS256 and a library will happily fall back to HS256, signs a forged token with your issuer's RSA *public* key used as an HMAC secret. Public keys are, correctly, public; using one as a symmetric secret is the bug. The fix is structural: your verification call should hardcode the one algorithm you expect and refuse anything else, never trust the token to self-report what algorithm was used to sign it.

## API keys: no user, no problem

An API key is a long, random, opaque string issued to an application or account, sent on every request (usually a header), and checked against a stored value. It carries no claims, no expiry baked in, no concept of "the user currently acting" — which is exactly the right shape for server-to-server integrations, third-party backends calling your API, or internal tooling, and exactly the wrong shape the moment you need to know which specific human triggered a request. `api-key-management-and-scopes`, the next lesson in this group, covers how to issue, store, scope, and rotate these safely — this lesson's job is just placing them correctly against the other three options.

## Basic Auth: still alive, mostly in the back office

HTTP Basic Auth sends `username:password` base64-encoded in an `Authorization` header, on every single request, full stop. Base64 is encoding, not encryption — it's as readable as plain text to anyone who intercepts it — so Basic Auth is only defensible over TLS, and even then it has no token lifecycle: no expiry, no scope, no way to revoke one client's access without rotating the underlying password for everyone using it. Where it survives today is internal service-to-service calls on a trusted network, legacy systems that predate anything better, and quick-and-dirty staging environments — not public-facing production APIs.

## Session cookies: state on the server, not in the token

A session-based API issues an opaque session ID on login, stores the actual session state (user ID, permissions, whatever else) server-side, and sends only that ID back to the client as a cookie. Every subsequent request, the server looks the ID up. This makes revocation trivial and instant — delete the session row and the user is logged out everywhere, immediately — at the cost of needing session storage shared across every instance of your API (a sticky-session hack doesn't survive a rolling deploy or an autoscaled fleet). It's the natural fit for same-origin browser apps where the browser already handles cookie storage and same-site transmission for you, and a poor fit for a public API with many independent client types, where "the API and the client share an origin" usually isn't even true.

## Matching the mechanism to the consumer

The pattern across all four: user-facing browser apps on your own origin lean toward session cookies (simple, instantly revocable, the browser does the work); mobile and third-party clients that need delegated, scoped access to a user's data lean toward the OAuth2/OIDC tokens from the previous lesson, which are typically JWTs under the hood; pure server-to-server integrations lean toward API keys; and Basic Auth stays in its lane — internal, trusted, TLS-only. `authorization-models-rbac-abac-and-beyond`, next in this group, picks up once you've established *who* is calling and turns to the separate question of *what* they're allowed to do once they're in.
