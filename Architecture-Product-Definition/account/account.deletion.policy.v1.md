# Account Deletion Policy V1

**Status:** Canonical
**Version:** V1
**Domain:** Account
**Parent:** `account/account.model.specification.v1.md`

---

# 1. Purpose

This document defines what happens to every account-owned entity when a Minime account is deleted.

Account deletion is initiated by the account owner through the Account Management System.

This document defines ownership, cascade order, and terminal outcome for every entity. It does not define the implementation mechanism, scheduling, or the UI flow.

---

# 2. Core Principle

Account deletion follows the platform Hard Delete philosophy.

Every entity owned by the account is either immediately deleted or immediately scheduled for deletion.

No entity associated with a deleted account remains permanently active.

---

# 3. What Triggers Deletion

Account deletion is an explicit user action.

The account owner initiates deletion through account settings.

No automated system may delete an account without an explicit user instruction.

---

# 4. Cascade Order

**Governing decision:** This order was promoted from the approved implementation (`implementation/10-background-jobs.md` — Job 7, "Account Deletion Cascade") to replace an earlier ordering that was never implemented and would have broken idempotency on job retry. See `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` — APD-015 for the full engineering rationale.

The following order must be respected to maintain both referential integrity and idempotency (safe re-execution after a partial failure):

```text
Step 1 — Sessions (invalidated synchronously; durably guaranteed by this cascade as a catch-all)
Step 2 — Out Link ID Collection (read-only; records which Out Links exist before anything is deleted)
Step 3 — Analytics Events (Hard Delete)
Step 4 — Out Links (Hard Delete)
Step 5 — Profile Content, Blocks, and their Binary Assets (Hard Delete, in that internal order: Blocks, then Image Assets, then the Profile Content record itself, including its Appearance State section)
Step 6 — Connected Accounts (Hard Delete)
Step 7 — AI Analysis Sessions (Hard Delete)
Step 8 — QR Code Record and its SVG Asset (Hard Delete)
Step 9 — Account Record Status (set to `deleted`)
```

**Why Analytics Events are deleted before Out Links (Step 3 before Step 4):** Analytics Events reference Out Link identifiers. If the cascade fails and retries after Out Links have already been deleted but Analytics Events have not, the retry would have no way to find which Analytics Events to delete (their only reference, the Out Link, would already be gone). Deleting Analytics Events first, using the Out Link IDs collected read-only in Step 2, guarantees that a retry at any point produces the same end state: if Out Links are already deleted, Step 2's collection returns an empty list and Step 3 becomes a safe no-op, rather than an unrecoverable gap.

**Why Blocks are deleted before the Profile Content record (within Step 5):** Blocks reference `account_id` directly and are deleted first; the Profile Content record (including its embedded Appearance State) is deleted last within this step, after any Image Assets its Blocks referenced have already been cleaned up. This ordering has no observable referential-integrity requirement in either direction (both are Hard Delete, keyed by `account_id`, with no foreign key between them) — it is a deliberate, arbitrary-but-fixed order chosen for one reason only: consistency with the rest of the cascade's "leaf data before container data" pattern, not an idempotency requirement, and this dependency is stable and does not require its own separate cascade steps.

**Why AI Analysis Sessions are deleted before the QR Code record (Step 7 before Step 8):** There is no referential or idempotency dependency between these two steps — Analysis Sessions and the QR Code record are both scoped only by `account_id` and do not reference each other. The QR Code record and its SVG asset are deleted last among the account-owned entities so that `/qr/{qr_code_id}` continues resolving to the (still momentarily existing) account record for as long as possible during the cascade, minimizing the window in which a QR scan could hit a partially-deleted account. This is a deliberate, fixed order; it does not need to be duplicated as reasoning at each retry.

Audit Logs are not deleted. They are retained per audit logging policy.

The username is not released. It remains permanently reserved.

---

# 5. Entity Deletion Rules

---

## Sessions

**Treatment:** Immediate invalidation.

All active sessions for the account are invalidated at the moment deletion is confirmed.

The user is signed out of all clients.

No new session may be created for a deleted account.

---

## Analytics Events

**Treatment:** Hard Delete.

All analytics events associated with the account's Out Links are permanently deleted, using a read-only collection of Out Link IDs taken *before* Out Links themselves are deleted (Step 2 in the Cascade Order above). This ordering — Analytics Events before Out Links — is deliberate: it is what makes the cascade safely retryable (see "Why Analytics Events are deleted before Out Links" in Section 4).

Analytics events reference Out Link identifiers only. They contain no personally identifying information. They are deleted because their referenced Out Links are about to no longer exist.

---

## Out Links

**Treatment:** Hard Delete.

All Out Links owned by the account are permanently deleted regardless of their current lifecycle state (Active, Archived), after Analytics Events referencing them have already been removed.

---

## Profile Content, Blocks, and Appearance State

**Treatment:** Hard Delete (single cascade step; see Section 4, Step 5).

All of the following are permanently deleted together, in this internal order: Blocks first, then the Profile Content record (which includes the Appearance State as an embedded section, not a separate entity):

* All blocks and all block types (avatar, name, bio, image, button, social icons, divider, title, textbox) — content and ordering references are deleted
* Display Name, Bio, Avatar asset reference, Contact information (Profile Content's canonical fields)
* Appearance State: selected theme reference and (once V2 theme customization exists) any customization values — see `appearance/appearance.state.specification.v1.md`. Appearance State is not an independent entity; it is a JSONB section (`appearance_config`) inside the Profile Content record, and is deleted as part of deleting that record, not as a separate step.

Theme definitions in the Theme Catalog are not affected by this step. Those belong to the Appearance system as a shared platform resource, not to the account.

---

## Connected Accounts

**Treatment:** Hard Delete.

All connected social accounts owned by the account are permanently deleted.

Each connected account record is deleted individually.

---

## AI Analysis Sessions

**Treatment:** Hard Delete.

All Analysis Session records owned by the account are permanently deleted, regardless of status (completed or failed).

There is no separate "accepted AI decision" entity in V1. Analysis Sessions are stored reports only; an accepted suggestion already exists as ordinary Product Domain data owned by the responsible domain (e.g. an accepted bio is already part of Profile Content, deleted in that step, before this one). Analysis Sessions are owned by the AI Platform and scoped to the account. They are deleted as part of the account data cascade, before the QR Code record (see Section 4, Step 7).

---

## QR Code Records

**Treatment:** Hard Delete. Last account-owned entity deleted in the cascade (Section 4, Step 8), immediately before the Account Record status itself changes.

The account's QR Code record (keyed by `account_id`) is permanently deleted.

The generated SVG QR asset is deleted via the Storage Platform as part of this same step (not a separate "Binary Assets" step — see below).

After deletion, `/qr/{qr_code_id}` must no longer resolve to an active profile.

---

## Binary Assets

**Treatment:** Physical deletion via Storage Platform, distributed across the steps above rather than as a separate final step.

Binary assets owned by the account are physically deleted from Storage at the point in the cascade where their owning record is deleted, not gathered into one final catch-all step:

* Avatar image and uploaded Image Assets — deleted as part of the Profile Content / Blocks step (Section 4, Step 5)
* Generated QR SVG asset — deleted as part of the QR Code Records step (Section 4, Step 8)

The Data Platform initiates each deletion by removing the asset reference at the same time as the owning record. The Storage Platform executes the physical deletion and, independently, detects and cleans up any orphaned assets that escape explicit deletion (e.g. from a prior failed attempt) per its standard orphan cleanup process — this is a defense-in-depth backstop, not the primary deletion path.

---

## Account Record Status

**Treatment:** Status set to `deleted`. Record retained.

The account record itself is not physically removed from the system.

The account record status is set to `deleted`.

The account identifier (`account_id`) is retained permanently to preserve audit log referential integrity.

The username field is retained as a permanently reserved username.

---

# 6. Audit Logs

**Treatment:** Retained.

Audit logs are not deleted when an account is deleted.

Audit logs are retained for the remainder of the 12-month period defined by the audit logging policy.

After the retention period expires, audit logs are deleted per standard audit log retention enforcement.

Audit logs are retained for operational and security integrity, not for user benefit. The account owner's right to deletion does not require audit log removal because audit logs contain system activity records, not user content.

---

# 7. Username

**Treatment:** Permanently reserved.

Deleted account usernames are not released in V1.

The username remains in `taken` state indefinitely.

No other user may claim a deleted account's username.

Username release policy may be introduced in a future version.

---

# 8. Public Profile

**Treatment:** Immediately inaccessible.

Once an account is deleted, the public profile access policy returns 404 for all requests to `/{username}`.

No rendering occurs. No cached profile is served beyond the CDN TTL defined by the cache policy.

---

# 9. Ownership Summary

| Order | Entity | Treatment | Owner of Decision |
|---|--------|-----------|-------------------|
| 1 | Sessions | Invalidated immediately | Authentication System |
| 2 | (Out Link ID collection) | Read-only | Out Links Domain |
| 3 | Analytics Events | Hard Delete | Data Platform |
| 4 | Out Links | Hard Delete | Data Platform |
| 5 | Blocks, Profile Content (incl. Appearance State), their Binary Assets | Hard Delete / Physical deletion | Data Platform / Storage Platform |
| 6 | Connected Accounts | Hard Delete | Data Platform |
| 7 | AI Analysis Sessions | Hard Delete | AI Platform |
| 8 | QR Code Record and its SVG Asset | Hard Delete / Physical deletion | Data Platform / Storage Platform |
| 9 | Account Record | Status → deleted | Data Platform |
| — | Username | Permanently reserved | Data Platform |
| — | Audit Logs | Retained 12 months | Audit Logging Policy |

This table's order column matches Section 4's Cascade Order exactly. Username and Audit Logs are not part of the deletion cascade itself (the username is a field on the Account record retained by Step 9; Audit Logs are never touched by this cascade at all).

---

# 10. Irreversibility

Account deletion is permanent.

There is no restoration capability in V1.

There is no grace period in V1.

Deletion is final.

---

# 11. What Deletion Does Not Affect

Account deletion does not affect:

* Theme definitions in the Theme Catalog (shared platform assets)
* Other users' accounts
* Other users' analytics events
* Platform-level configuration

---

# 12. V1 Non-Goals

This policy does not define:

* The implementation mechanism for cascade operations
* Soft-delete grace periods (not supported in V1)
* Data export before deletion
* Third-party data deletion requests
* GDPR Article 17 legal compliance specifics

Those belong to operational policy and legal compliance review, not product architecture.
