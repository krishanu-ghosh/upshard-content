---
title: "Authorization Models: RBAC, ABAC, and Beyond"
description: The authorization-model taxonomy — RBAC, ABAC, PBAC, ReBAC, MAC, DAC — and the signals that tell you which one fits the access-control problem in front of you.
roadmap: api-design
node: authorization-models-rbac-abac-and-beyond
group: authentication-and-authorization
video:
  provider: youtube
  id: "byI_Jjb0c6c"
  start: 0
  end: null
creator:
  name: Nordic APIs
  channel_url: https://www.youtube.com/@nordicapis
duration_seconds: 1132
level: advanced
prerequisites:
  - jwt-api-keys-and-session-auth
takeaways:
  - "RBAC (role → permissions) is simple to reason about and audit — \"what can the Editor role do\" is one query — but it's coarse: real systems accumulate roles like Editor-of-project-X-but-not-Y until the role table is really encoding per-resource access with extra steps."
  - "ABAC evaluates a policy against attributes of the user, the resource, and the environment at request time (\"an engineer can edit a document they created, during business hours, from a company network\"), which is far more flexible than RBAC but harder to audit — \"what can this user do\" requires evaluating the policy against every possible context instead of reading a role table."
  - "PBAC is policy-as-code: authorization rules live in a versioned, testable policy file (often Rego, evaluated by an engine like Open Policy Agent) rather than scattered through application code — it's less a separate model than a way of *implementing* ABAC-style rules so they're reviewable and decoupled from the service that enforces them."
  - "ReBAC determines access by walking a graph of relationships — \"can user X view document Y\" resolves through chains like X is a member of team T, T is a collaborator on folder F, Y is inside F — which is what Google Docs' and GitHub's real sharing models actually run on, not a flat role or a static attribute check."
  - "Google's Zanzibar paper (2019) is the reason ReBAC is having a moment: it's the authorization system behind Google Docs, Drive, and Calendar, serving relationship checks at a scale and consistency guarantee (\"if I just removed your access, you cannot still see the document a moment later\") that pushed open-source descendants like OpenFGA and SpiceDB into existence."
  - "MAC and DAC are the oldest two models: MAC enforces a centrally-set policy no resource owner can override (classified document clearance levels), DAC lets the resource's owner decide who gets access (classic Unix file permissions, chmod) — most consumer and SaaS systems are DAC-flavored by default; MAC shows up where compliance, not convenience, is the design constraint."
check_questions:
  - q: Why does RBAC tend to degrade into "role explosion" as a system's access requirements get more specific?
    a: RBAC has no native concept of per-resource context — if access actually needs to vary by which specific project, document, or record is involved, the only way to express that in pure RBAC is to mint a new role per combination (Editor-of-Project-A, Editor-of-Project-B...), and the role table grows to encode exactly the resource-level distinctions RBAC was supposed to abstract away.
  - q: What's the practical difference between ABAC and PBAC, given that both are often described as "policy-based"?
    a: ABAC is the model — access decisions are a function of attributes on the user, resource, and environment, evaluated at request time. PBAC describes *how* that evaluation gets implemented — as versioned, declarative policy code (commonly Rego run through Open Policy Agent) separated from the application, rather than attribute checks hand-written into each service's business logic. Most real PBAC systems are ABAC under the hood, expressed as policy-as-code.
  - q: How does ReBAC answer "can user X view document Y" differently from RBAC or ABAC, and why does that matter for something like a shared folder?
    a: ReBAC walks a graph of relationships rather than checking a role or evaluating a static rule — X's access to Y is derived transitively through intermediate relationships (X belongs to team T, T can access folder F, Y lives in F), so access to a newly added document inside an already-shared folder is correct automatically, without anyone having to explicitly grant X access to that specific new document.
  - q: Why would a system choose MAC over DAC even though DAC is simpler and lets resource owners manage their own access?
    a: DAC trusts every resource owner's individual access decisions, which is fine until a compliance or security requirement needs access to be enforced consistently regardless of what any individual owner decides — MAC centralizes that policy so no owner can accidentally (or deliberately) grant access that violates a clearance level or regulatory rule, trading owner flexibility for guaranteed consistency.
deeper:
  - label: "Explain Access Control Models | Discretionary DAC, Mandatory MAC, RBAC, Rule, Attribute, Risk based"
    url: "https://www.youtube.com/watch?v=PjZH-CO1hX0"
related_formulas: []
---

## Authentication answers who, authorization answers what

`jwt-api-keys-and-session-auth` establishes who's calling your API. This lesson is entirely about the next question: once you know who they are, what are they actually allowed to do? That question has a real taxonomy of answers, and picking the wrong one is a genuinely common and genuinely expensive mistake — usually discovered the hard way, as a broken object-level authorization bug (`owasp-api-security-top-10`, later in this roadmap, covers that failure mode directly). This lesson is new ground nothing else in Upshard's catalog covers: the six real authorization models, what each one is actually good at, and the signals that tell you which one fits.

## RBAC: roles, and where they stop being enough

Role-based access control is the default everyone reaches for first, and for good reason: define a handful of roles (Admin, Editor, Viewer), assign permissions to each role instead of to individual users, and assign users to roles. "What can an Editor do" is a single lookup, which makes RBAC easy to audit and easy to explain to a non-engineer during a compliance review.

It breaks down the moment access needs to vary *per resource* rather than globally. "Alice can edit Project A but only view Project B" isn't expressible with one global Editor role — you end up minting `Editor-of-Project-A` and `Editor-of-Project-B` as separate roles, and the role table starts encoding exactly the resource-level granularity RBAC was supposed to abstract away. This is "role explosion," and it's the tell that a system has outgrown pure RBAC.

## ABAC: policy over attributes

Attribute-based access control evaluates a rule against attributes of three things at request time: the user (department, clearance, team), the resource (owner, sensitivity, project), and the environment (time of day, network, request origin). "An engineer can edit a document they created, during business hours, from the company network" is an ABAC rule — none of it is expressible as a static role assignment, because the decision depends on facts only known at request time.

The cost is auditability. With RBAC, "what can this user do" is one query. With ABAC, answering that honestly means evaluating the policy against every resource and every context the user might ever touch — there's no single table to read. ABAC trades a simple audit story for real flexibility, and that trade is worth it exactly when access genuinely depends on context RBAC can't express.

## PBAC: attribute logic as reviewable code

Policy-based access control isn't a competing model so much as an implementation discipline for ABAC-shaped rules: instead of attribute checks scattered through `if` statements across a dozen services, the policy lives in one place, as declarative, versioned code — commonly written in Rego and evaluated by an engine like Open Policy Agent (OPA), which services call out to for a yes/no decision rather than reimplementing the logic themselves. The win is that a policy change becomes a reviewable pull request against one file instead of a coordinated code change across every service that happens to enforce that rule.

## ReBAC: access as a graph query

Relationship-based access control is the newest of these to get mainstream attention, and it answers "can user X view document Y" by walking a graph of relationships rather than checking a role or a static attribute: X is a member of team T, T is a collaborator on folder F, Y lives inside F — access to Y follows transitively, with nobody having explicitly granted X access to that specific document. This is what actually powers sharing in Google Docs and GitHub: add a document to an already-shared folder, and everyone with folder access can see it immediately, because access was never stored per-document in the first place — it's derived from the graph at query time.

Google's 2019 Zanzibar paper is the reason this has a name and a body of open tooling now: it describes the authorization system behind Google Docs, Drive, and Calendar, built to answer relationship queries at enormous scale while guaranteeing that a just-revoked permission can't still return "allowed" a moment later (a real consistency problem once you're serving billions of checks from replicated data). OpenFGA and SpiceDB are the open-source descendants, letting a team adopt Zanzibar's model without Google's internal infrastructure.

## MAC and DAC: the two oldest answers

Discretionary access control is the model you already know even if you've never heard the term: the resource's owner decides who else gets access, the way Unix file permissions work — you own a file, you `chmod` it, done. It's simple and it's the default for most consumer software and SaaS products, because "the person who created this thing controls who else sees it" matches how people actually think about ownership.

Mandatory access control removes that discretion: access is governed by a centrally set policy — classification levels, compliance rules — that no individual resource owner can override, even if they want to grant access to someone. It's the model behind military and government clearance systems, and it shows up in civilian software wherever a regulatory or security requirement needs access enforced consistently regardless of what any one person decides, which is a deliberately narrower and more rigid bar than DAC's "the owner decides."

## Reading the signals

None of these six is "more advanced" than the others in some universal sense — a small internal admin tool with three roles genuinely doesn't need ReBAC, and a document-sharing product with nested folders and team hierarchies genuinely can't be modeled cleanly in RBAC alone. The real skill is reading which signal dominates: if access maps cleanly to a small set of job functions, RBAC; if it depends on runtime context the role system can't see, ABAC (implemented as PBAC if you want it reviewable and centralized); if access is really about who's connected to what through a web of ownership and membership, ReBAC; and if a compliance regime demands access nobody — not even the resource owner — can override, MAC. `api-key-management-and-scopes`, next in this group, is the practical following question once you've picked a model: how do you actually issue and scope the credentials that carry this authorization at the API layer.
