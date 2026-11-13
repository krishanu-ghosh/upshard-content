---
title: "OAuth2 and OIDC in Practice"
description: Wiring OAuth2 into an API you build — which grant type fits which client, how access and refresh tokens rotate, and how OpenID Connect layers identity on top.
roadmap: api-design
node: oauth2-and-oidc-in-practice
group: authentication-and-authorization
video:
  provider: youtube
  id: "8aCyojTIW6U"
  start: 0
  end: null
creator:
  name: OktaDev
  channel_url: https://www.youtube.com/@OktaDev
duration_seconds: 2001
level: advanced
prerequisites: []
takeaways:
  - "OAuth2 is delegated *authorization* — a client gets a scoped token to act on a resource owner's behalf — not identity. If you need to know who the user is, that's OpenID Connect riding on top, not OAuth2 itself."
  - "Authorization code + PKCE is the modern default for every client that involves a user, public or confidential: single-page apps, mobile apps, and traditional server-rendered apps alike. PKCE isn't a mobile-only patch — the OAuth 2.1 draft requires it everywhere the authorization code flow is used."
  - "Client credentials is for service-to-service calls with no user in the loop at all — a cron job, a backend worker, a billing service calling your API. The 'user' being authorized is the client application itself."
  - "Device code exists for devices that can display a code but can't easily accept text input or run a full browser redirect — smart TVs, CLIs, IoT consoles — by moving the actual login to a second device the user already has."
  - "Implicit and resource-owner-password are both deprecated: implicit returned access tokens straight in the URL fragment with no way to prove the receiving client's identity, and resource-owner-password requires the client to directly handle the user's raw password, defeating the entire point of delegation."
  - "A refresh token should be rotated on every use — issue a new refresh token alongside the new access token and invalidate the old one — so a stolen refresh token that gets used even once is detectable and the window for replay is a single use, not the token's full lifetime."
check_questions:
  - q: Why is PKCE required for authorization code flow even in a traditional server-rendered web app, not just mobile and SPA clients?
    a: PKCE protects against authorization code interception regardless of client type — the OAuth 2.1 draft folds it into the authorization code flow universally rather than treating it as a public-client-only patch, since the attack it defends against (an intercepted code being redeemed by someone other than the client that requested it) isn't unique to mobile or browser-based apps.
  - q: An API needs to let a billing microservice fetch usage data from another internal API with no human involved at any point. Which OAuth2 grant type fits, and why not authorization code?
    a: Client credentials — the billing service authenticates as itself with its own client ID and secret and receives a token scoped to its own identity. Authorization code requires a resource owner to redirect through a login and consent screen, which doesn't exist when there's no human in the flow.
  - q: What does an OIDC ID token give you that a plain OAuth2 access token doesn't?
    a: An ID token is a JWT that asserts who authenticated and when, signed by the authorization server and intended for the client itself to read — an access token is opaque-by-design to the client and only proves the bearer is authorized to call a resource server, carrying no guaranteed claims about the end user's identity.
  - q: Why can't you safely trust an access token's claims without calling the identity provider, if the token is a JWT?
    a: You can, as long as you verify the signature against the identity provider's public key first — a JWT's claims are only trustworthy once the signature check passes; an unverified JWT is just a base64-encoded string anyone could have crafted, which is exactly the mistake `jwt-api-keys-and-session-auth` covers in the broader JWT-validation context.
deeper:
  - label: "An Illustrated Guide to OAuth and OpenID Connect"
    url: "https://www.youtube.com/watch?v=t18YB3xDfXI"
related_formulas: []
---

## Delegated authorization, not identity

Every confusion about OAuth2 traces back to one fact: it was designed to solve "let app A act on my behalf against app B's API," not "prove who I am." A user grants a client limited, scoped access to their resources without handing over their password — that's the entire original problem. Identity got bolted on top later, as OpenID Connect, because the industry kept misusing OAuth2 for login anyway and it was better to standardize that misuse than leave everyone inventing their own. If you remember nothing else: an OAuth2 access token answers "is the bearer allowed to do this," and only an OIDC ID token answers "who is this."

This lesson assumes you already know why your API needs real authentication and authorization distinct from one-off shared secrets — `structured-error-handling-and-rfc-7807` covers how your API should fail when a token is missing or invalid — and it deliberately skips the architecture-level question of *why* an organization federates identity across many services. `system-design`'s `authn-and-federated-identity-at-scale` owns that ground: SAML versus OIDC at the org level, why you'd stand up a central identity provider at all. This lesson is the practitioner's question: you're building an API, a client needs to call it, which flow do you actually implement.

## Picking a grant type

OAuth2 offers several grant types because "a client needs a token" means different things depending on what kind of client it is and whether a human is involved.

**Authorization code + PKCE** is the default for anything a user logs into — web app, native mobile app, or single-page app. The client redirects the user to the authorization server, the user authenticates and consents, and the authorization server redirects back with a one-time code the client exchanges for tokens. PKCE (Proof Key for Code Exchange) adds a client-generated secret — a `code_verifier` hashed into a `code_challenge` sent up front — so that even if the authorization code is intercepted in transit, the attacker can't redeem it without the verifier only the original client holds. This used to be framed as a "public client" patch for apps that can't keep a client secret (mobile, SPA), but OAuth 2.1 makes it mandatory for every authorization code flow, confidential clients included, because the interception risk isn't specific to public clients.

**Client credentials** is for machine-to-machine calls: a backend job, a billing service, a scheduled worker calling your API with no user anywhere in the picture. The client authenticates with its own client ID and secret directly against the token endpoint and gets back a token scoped to itself, not to any resource owner. This is the grant type you reach for when "who is making this request" is answered by "this specific service," full stop.

**Device code** solves a narrower but real problem: a device with a screen but no comfortable way to type a password or run a full browser-based redirect — a smart TV, a CLI tool, a hardware console. The device displays a short code and a URL; the user opens that URL on a phone or laptop, enters the code, and authenticates there, while the device polls the token endpoint in the background until the user finishes. You've seen this as "visit tv.youtube.com/activate and enter XXXX-XXXX" — that's a device code flow.

**Implicit** and **resource-owner-password** are both deprecated, and you shouldn't implement either in a new API. Implicit returned the access token directly in the URL fragment after redirect, with no code-exchange step — meaning no way to bind the token to the client that requested it, and a token sitting in browser history and server logs along the way. Resource-owner-password has the client collect the user's actual username and password and forward them to the authorization server, which defeats the point of OAuth2 entirely: the whole idea was that the client never sees the password.

## Access tokens, refresh tokens, and rotation

An access token is short-lived — minutes to a couple of hours — and sent with every API call. A refresh token is long-lived and used only to get a new access token when the old one expires, without forcing the user through the login screen again. Keeping access tokens short-lived limits the blast radius of a leaked token; keeping the refresh token separate means the thing an attacker would need to steal for *persistent* access is handled with more care (never sent to a resource server, only ever to the token endpoint).

**Refresh token rotation** is the detail that separates a toy implementation from a production one: every time a refresh token is used, issue a new one and invalidate the old one, rather than letting the same refresh token work indefinitely. If an attacker ever steals a refresh token and uses it, the legitimate client's next refresh attempt will fail — the token it's holding was just invalidated by the attacker's use — which is your signal that something is wrong and a flag you can act on (revoke the whole token family, force re-authentication).

## OIDC: identity on top of authorization

OpenID Connect adds exactly two things OAuth2 doesn't have: the **ID token**, a JWT asserting who authenticated and when, signed by the authorization server and meant to be read by the client itself (unlike the access token, which the client should treat as opaque); and the **`/userinfo` endpoint**, which a client can call with its access token to fetch the authenticated user's profile claims — email, name, picture — without the authorization server having to cram all of that into the ID token up front. Requesting the `openid` scope alongside whatever resource scopes you need is what turns a plain OAuth2 request into an OIDC request and gets you an ID token back in addition to the access token.

Once you have a token — whichever grant produced it — `jwt-api-keys-and-session-auth` is where this roadmap covers what you actually do with it on the receiving end: validating the JWT correctly, and how it stacks up against API keys, Basic Auth, and session cookies for the API-consumer shapes that don't need full OAuth2 at all.
