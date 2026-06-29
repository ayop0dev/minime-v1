# Minime V1 — Implementation Reference

## Status

**READY FOR IMPLEMENTATION.**

Implementation repository is complete and frozen. All 13 specification documents have passed integrity audit. Phase 03 (Backend Implementation) may begin.

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

- Supported methods in V1: Email OTP and Google Sign-In only.
- Phone OTP is not supported in V1.
- SMS is not supported in V1.
- OAuth is not implemented in V1 (ConnectedAccounts does not use OAuth).

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
- Implement phone OTP or SMS verification
- Implement OAuth flows or token storage
- Add lifecycle states not defined by the frozen architecture
- Add Product Domains not defined by the frozen architecture
- Add Platform Services beyond: Data, Storage, Events, AI
- Duplicate business logic across domains
- Allow Platform Services to own Product Domain decisions
