---
title: "API Key Management and Scopes"
description: How to issue, store, scope, rotate, and revoke API keys safely, and why keys matter as a rate-limiting identity even for callers who barely need authentication.
roadmap: api-design
node: api-key-management-and-scopes
group: authentication-and-authorization
video:
  provider: youtube
  id: "MZaOYPtnPzw"
  start: 0
  end: null
creator:
  name: Zuplo
  channel_url: https://www.youtube.com/@Zuplo
duration_seconds: 1161
level: intermediate
prerequisites:
  - authorization-models-rbac-abac-and-beyond
takeaways:
  - "Store API keys the same way you store passwords: hash them (a fast cryptographic hash like SHA-256 is fine here, unlike passwords, because a good API key already has enormous entropy and isn't guessable by a dictionary attack) and never keep the plaintext value anywhere after the one moment you issue it."
  - "A key only needs to be shown to its owner once, at creation — this is 'irretrievable by design': if a customer loses their key, the only correct recovery path is issuing a new one, the same way you'd never recover a user's plaintext password."
  - "Prefixing keys with a recognizable, structured pattern (`sk_live_...`, `sk_test_...`) isn't cosmetic — it lets you distinguish key types at a glance in logs, lets automated secret-scanning tools (like GitHub's) detect your leaked keys in public repos and auto-notify you, and lets you validate a key's shape before ever hitting the database."
  - "Rotation needs an overlap window, not a hard cutover: issue the new key, give both keys a period where they're valid simultaneously, let the integration update at its own pace, then revoke the old one — a hard cutover turns routine rotation into a guaranteed outage for anyone who hasn't updated yet."
  - "Scopes are least privilege applied to a single key rather than a whole account: a key scoped to `read:orders` can't delete anything even if the account it belongs to has full admin rights, which shrinks the blast radius of that one key leaking to exactly what it was allowed to do."
  - "A key is also the natural identity to rate-limit against — `system-design`'s `rate-limiting-algorithms` covers the actual algorithms (token bucket, sliding window, and the rest); the point here is narrower: per-key limits let you throttle one noisy or compromised integration without touching every other caller sharing the same account."
check_questions:
  - q: Why should an API key be hashed before storage even though, unlike a password, nobody is expected to remember or reuse it?
    a: Hashing protects against the same threat either way — a database breach. If keys are stored in plaintext, a single leaked database exposes every live key immediately and silently; hashed, a breach exposes nothing usable without reversing a one-way function, and you still have the irretrievable-by-design story (issue new, don't recover old) whether or not the breach ever happened.
  - q: What real problem does a rotation overlap window solve that a hard key cutover doesn't?
    a: An integration partner's deploy cadence is outside your control — they can't necessarily update their stored key the instant you rotate it. An overlap window where both the old and new key validate keeps their requests succeeding while they update on their own schedule; a hard cutover makes every caller who hasn't updated yet fail starting the instant you rotate, turning routine key hygiene into an unplanned outage for someone else's system.
  - q: A partner integration only needs to read order data, never write or delete anything. What does scoping the key to `read:orders` protect against that a full-access key on the same account wouldn't?
    a: If that specific key leaks — in a log, a committed `.env` file, a compromised CI pipeline — the damage is bounded to reading order data, regardless of what else the owning account is allowed to do. An unscoped key tied to a full-admin account turns the same leak into full account compromise instead of a contained read-only exposure.
  - q: Besides authenticating the caller, why does rate-limiting by API key matter even for an endpoint that doesn't strictly require authentication?
    a: A key gives you a stable per-caller identity to count requests against, so one integration sending a burst of traffic (buggy retry loop, scraper, or genuine spike) can be throttled without affecting every other caller — rate-limiting by IP alone breaks down behind shared NATs and proxies, and rate-limiting globally punishes every consumer for one bad actor's behavior.
deeper:
  - label: "Permissions, Privileges and Scopes - What's the Difference?"
    url: "https://www.youtube.com/watch?v=z2Ma0ZsEF7Y"
related_formulas: []
---

## Why API keys still exist next to OAuth2

`jwt-api-keys-and-session-auth` already placed API keys against JWTs, Basic Auth, and session cookies: a key is the right shape for a server-to-server caller that doesn't need per-user context. This lesson picks up one level deeper — once you've decided a key is the right primitive, how do you actually run a key system without it becoming the next incident report. Stripe, GitHub, Resend, and most API-first companies default to keys for exactly this reason: they're simple to issue, simple for a consumer to use (one header, no OAuth dance), and the operational discipline around them is well understood if you follow a few rules consistently.

## Issuance and storage: treat keys like passwords

The single most important rule is the one teams skip because it feels unnecessary: **hash the key before storing it**, the same discipline you'd apply to a user's password. A plaintext key sitting in your database means one database breach instantly compromises every live key your API has ever issued, silently, with no way for affected customers to even know until damage shows up. Hash it with a standard cryptographic hash (SHA-256 is sufficient here — unlike a password, a properly generated API key already has enough entropy that slow, salted password-hashing schemes like bcrypt aren't buying you anything extra against a dictionary attack) and store only the hash plus enough metadata — a short prefix, creation date, owner — to manage the key without ever needing the full value again.

This is why key systems are **irretrievable by design**: you show the customer the full key exactly once, at the moment of creation, and never again. If they lose it, the correct answer is "issue a new one," not "look it up" — there's no legitimate reason your system should ever need to retrieve a plaintext key after the moment it was generated, and building a "show my key again" feature defeats the entire point of hashing it in the first place.

A well-designed key also carries a structured, recognizable prefix — Stripe's `sk_live_...` and `sk_test_...` are the canonical example. This isn't decoration: a prefix lets you tell live and test keys apart at a glance in logs and error reports, lets you reject malformed keys before a database round trip, and — critically — lets automated secret-scanning (GitHub's push-protection and secret-scanning partner program chief among them) recognize your key format in a public commit and notify both you and the customer before an attacker finds it first.

## Rotation: overlap, don't cut over

Rotation policy is where key systems most often go wrong in practice, not in principle. The principle — keys should be rotated periodically, and immediately after any suspected leak — is uncontroversial. The failure mode is rotating by hard cutover: generate a new key, invalidate the old one instantly, and now every integration that hasn't updated yet starts failing the moment you rotate, which makes routine hygiene feel to your customers like you just broke their integration without warning.

The fix is an overlap window: issue the new key, keep the old one valid for a defined grace period (days, sometimes weeks depending on how often your customers deploy), notify the key's owner that rotation is happening and by when the old key dies, then revoke the old key once the window closes or once you can confirm traffic has shifted to the new one. This turns rotation from a breaking change into a scheduled migration your customers control the timing of, within your window.

## Scopes: least privilege per key, not per account

A single account issuing one all-powerful key for every integration it runs is the key-management equivalent of giving every service the root database password. Scopes fix this by attaching specific, limited permissions to the key itself, independent of what the owning account is otherwise allowed to do — a key scoped to `read:orders` can't touch billing or delete anything, even on a full-admin account, because the scope check happens against the key's own grant, not the account's ceiling. This is `authorization-models-rbac-abac-and-beyond` applied at the smallest possible unit: instead of one authorization decision per account, you get one per credential, which shrinks the blast radius of any single key leaking to exactly what that key was scoped to do and nothing more. In practice this means designing your scope vocabulary deliberately — `read:orders`, `write:orders`, `read:customers` — rather than a single boolean "has API access," so that a partner integration that only ever needs to read order status can be issued a key that structurally cannot do anything else.

## Revocation, and the key as a rate-limiting identity

Revocation should be immediate and simple: flag the key (or delete its hash) and every subsequent request carrying it fails authentication on the next lookup — there's no token expiry to wait out the way there can be with a long-lived JWT. Build revocation in from day one rather than bolting it on later; "we can't actually turn off one customer's key without affecting others" is a design flaw, not an edge case, once you're running a real key system.

One more reason keys exist beyond pure authentication: a key is a stable, per-caller identity to count requests against. `system-design`'s `rate-limiting-algorithms` covers the actual mechanics — token bucket, sliding window, and the rest — but the relevant point here is narrower: rate-limiting by API key lets you throttle one specific noisy, buggy, or compromised integration without collateral damage to every other caller on the same account or the same IP range. A shared account with five integrations and one misbehaving script shouldn't mean throttling the other four — per-key limits are what make that kind of surgical response possible.
