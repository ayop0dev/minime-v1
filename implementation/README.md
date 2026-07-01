# Minime V1 — Implementation Reference

## Status

**Implementation specifications are approved for V1.**

The 20 critical/high implementation blockers, the cross-domain conflict table, and the missing-contract inventory identified by the prior blockers audit have been addressed by architectural clarification (not redesign). The specific implementation-enabling engineering decisions produced by that resolution pass — and every decision approved since — are recorded in the canonical registry:

```
ARCHITECTURE_PR_APPROVAL_DECISIONS.md
```

at the repository root. Those decisions are implementation-enabling only: they make the frozen architecture concrete enough to build against (entities, background jobs, deployables, repository contracts, transaction coordinators, advisory locks, unique indexes, security boundaries). They are not product redesign, do not expand product scope, and do not introduce a new Product Domain. Any future addition of this kind requires a new architecture approval recorded in that registry before implementation proceeds — see "Post-Freeze Clarification Policy" below.

Specification documents in this directory are not self-certifying; do not restate "ready for implementation" here without re-running consistency verification against the current state of the repository.

---

## Authority

The frozen Product Architecture is the only source of truth.

Every statement in this directory is traceable to:

```
Architecture-Product-Definition/docs/MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md
```

When any implementation statement conflicts with the frozen architecture, the architecture wins.

Do not modify the architecture folder.

---

## Post-Freeze Clarification Policy

The frozen architecture may receive implementation-enabling clarifications only.

A clarification:

- must remain minimal — it makes an already-approved rule concrete enough to implement, nothing more;
- must not expand product scope;
- must not redesign architecture, rename existing concepts, remove existing architecture, or introduce a new Product Domain.

If a clarification introduces any of the following, it must be explicitly recorded in `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` before or alongside its introduction:

- an entity
- a background job
- a deployable
- a repository contract
- a transaction coordinator
- an advisory lock
- a unique index
- a security boundary

A clarification of this kind that is not recorded in that registry is not approved for implementation, regardless of where else in the repository it appears.

---

## What This Directory Is

This directory translates the frozen Product Architecture and Platform Architecture into engineering specifications that direct production code.

Every specification describes HOW the approved architecture is built.

No specification redesigns WHAT the product is.

---

## Absolute Invariants

These are derived directly from the frozen architecture. Every implementation decision must be consistent with these invariants.

**Identity**

- Account is the single canonical persisted identity in Minime V1.
- There is no User entity in Minime V1.
- There is no Profile entity in Minime V1.
- There is no PublicProfile entity in Minime V1.
- There is no `profile_id` in Minime V1.
- `account_id` is the canonical ownership identifier across every table.

**Profile Content**

- Profile content (display name, bio, avatar, contact, appearance) belongs to Account.
- Profile content is stored in a `ProfileContent` table where `account_id` is both the primary key and the foreign key.
- Blocks reference `account_id`, not `profile_id`.
- Out links reference `account_id`, not `profile_id`.

**No Publishing Workflow**

- There are no draft, unpublished, scheduled, preview, or separate live profile states.
- Saving any profile change makes it publicly visible within ≤ 60 seconds (cache TTL).
- `ProfileService` has no `publishProfile` or `unpublishProfile` operation.

**Username**

- Username is a field on `Account`, not a separate persisted entity.
- Username is assigned during registration and is not editable in V1.
- `UsernameReservation` is a temporary record used only during registration flow. It is not Account-owned.

**Authentication**

- Supported providers in V1: Google Sign-In and Apple Sign-In only, each implemented as a backend-driven OAuth/OIDC authorization-code flow dispatched through a Provider Adapter (`AuthService.startOAuth` / `handleOAuthCallback`; see `authentication.policy.v1.md` — "Provider Adapter Architecture").
- Authentication is provider-based only. Minime never authenticates users directly.
- `provider` is constrained to `google` | `apple` in V1; no generic open-ended provider value exists. `provider` is always request data, never API route structure — there is no `/auth/{provider}/*` route of any kind. A future provider is added by implementing a new adapter and extending the allowlist, never by changing routes, service contracts, or the data model shape.
- Identity is `(provider, provider_subject)` — the only authentication identity key. `email` is optional, provider-returned profile metadata only — never the canonical identity key, never used for automatic account merge. There is no automatic and no manual account merge in V1.
- `AuthenticationIdentity` has no `display_name` field; provider-reported names live only inside `provider_profile`, generic non-authoritative metadata that never replaces Profile Content.
- Every account has at least one `AuthenticationIdentity`; the last remaining one can never be removed. A second provider may be explicitly linked (by an already-logged-in user) and later unlinked, addressed by `auth_identity_id`. `primary_provider`, if set, is a display preference only and never affects ownership or authorization.
- Email OTP is not supported in V1.
- Phone OTP, SMS, and WhatsApp OTP are not supported in V1.
- Password authentication, Facebook Login, X Login, and LinkedIn Login are not supported in V1.
- Connected Accounts (social links) do not use OAuth and are not authentication providers — this is a separate, unrelated boundary from Authentication's OAuth flows above, and it holds even if a future Authentication Provider Adapter targets a platform that is also a Social Accounts platform (e.g. Instagram, Facebook).

**Social Accounts vs Connected Accounts**

- The Social Accounts domain is a processing domain. It normalizes user-provided input and generates canonical URLs. It owns no persistent storage.
- The ConnectedAccount entity is the canonical persistent record for user-entered social links.
- ConnectedAccount stores: `platform`, `username`, `url`, `sort_order`.
- ConnectedAccount uses Hard Delete. There is no `deleted_at` on this entity.
- ConnectedAccount has no status model (no active/inactive/pending states).

**Rendering**

- Rendering is stateless. It never stores, owns, or modifies data.

**Analytics**

- Analytics is observational. It never modifies business entities.

---

## Directory Structure

```
implementation/
├── README.md                       ← invariants, reading order, rules
├── 01-technology-stack.md          ← approved technology decisions
├── 02-repository-structure.md      ← monorepo structure
├── 03-canonical-data-model.md      ← account-centric entity model + Prisma schema
├── 04-service-contracts.md         ← all domain service contracts
├── 05-api-contracts.md             ← global API rules + all domain endpoints
├── 06-event-contracts.md           ← internal event contracts
├── 07-validation-rules.md          ← validation layers with concrete values
├── 08-security-model.md            ← trust boundaries and protections
├── 09-caching-strategy.md          ← Redis caching rules
├── 10-background-jobs.md           ← async job model
├── 11-platform-services.md         ← Storage + AI platform contracts
├── 12-seo-and-integrations.md      ← cross-cutting capabilities
└── 13-implementation-plan.md       ← phased execution plan
```

---

## Reading Order

```
01-technology-stack
      │
      ▼
02-repository-structure
      │
      ▼
03-canonical-data-model        ← start here before any domain work
      │
      ▼
04-service-contracts
      │
      ▼
05-api-contracts
      │
      ▼
06-event-contracts
      │
      ▼
07-validation-rules
      │
      ▼
08-security-model
      │
      ▼
09-caching-strategy
      │
      ▼
10-background-jobs
      │
      ▼
11-platform-services
      │
      ▼
12-seo-and-integrations
      │
      ▼
13-implementation-plan
```

---

## Engineering Rules

**Must:**

- Use `account_id` as the ownership reference for all account-owned data
- Implement repositories as the only persistence access layer
- Implement business logic only inside Product Domain services
- Validate all external input before business execution
- Publish events only after successful persistence
- Keep PostgreSQL as the canonical source of truth
- Keep controllers thin (no business logic)
- Use Prisma through repositories only (never from controllers)

**Must Not:**

- Introduce a `User` model, table, or entity
- Introduce a `Profile` model, table, or entity
- Introduce `profile_id` as a foreign key anywhere
- Implement `publishProfile` or `unpublishProfile`
- Implement username editing operations
- Implement Email OTP, phone OTP, SMS, WhatsApp OTP, password authentication, Facebook Login, X Login, or LinkedIn Login
- Implement Connected/Social Accounts OAuth flows or provider token storage for social platforms (distinct from Authentication's Google/Apple OAuth, which is implemented per `authentication.policy.v1.md`)
- Persist any provider access token or refresh token (Authentication or Social) anywhere in V1
- Accept a `provider` value other than `google` or `apple` on any authentication endpoint, or use `email` as the canonical authentication identity
- Add lifecycle states not defined by the frozen architecture
- Add Product Domains not defined by the frozen architecture
- Add Platform Services beyond: Data, Storage, Events, AI
- Duplicate business logic across domains
- Allow Platform Services to own Product Domain decisions
