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

The following order must be respected to maintain referential integrity.

```text
Step 1 — Sessions
Step 2 — Out Links
Step 3 — Analytics Events
Step 4 — Profile Content
Step 5 — Blocks
Step 6 — Appearance State
Step 7 — Connected Accounts
Step 8 — QR Code Records
Step 9 — AI Analysis Sessions
Step 10 — Binary Assets
Step 11 — Account Record Status
```

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

## Out Links

**Treatment:** Hard Delete.

All Out Links owned by the account are permanently deleted regardless of their current lifecycle state (Active, Archived).

Out Links are deleted before analytics events to allow events to be deleted as part of the same cascade operation.

---

## Analytics Events

**Treatment:** Hard Delete.

All analytics events associated with Out Links deleted in the previous step are permanently deleted.

Analytics events reference Out Link identifiers only. They contain no personally identifying information. They are deleted because their referenced Out Links no longer exist.

---

## Profile Content

**Treatment:** Hard Delete.

All canonical profile fields are permanently deleted:

* Display Name
* Bio
* Avatar asset reference
* Contact information
* Block order references

---

## Blocks

**Treatment:** Hard Delete.

All blocks owned by the account are permanently deleted.

This includes:

* All blocks
* All block types (avatar, name, bio, image, button, social icons, divider, title, textbox)

Block content is deleted. Block ordering references are deleted.

---

## Appearance State

**Treatment:** Hard Delete.

The account's appearance configuration is permanently deleted:

* Selected theme reference
* Theme customization values
* Design token overrides

Theme definitions in the Theme Catalog are not affected. Those belong to the Appearance system, not the account.

---

## Connected Accounts

**Treatment:** Hard Delete.

All connected social accounts owned by the account are permanently deleted.

Each connected account record is deleted individually.

---

## QR Code Records

**Treatment:** Hard Delete.

The account's QR Code record (keyed by `account_id`) is permanently deleted.

The generated SVG QR asset is deleted via Storage Platform as part of binary asset deletion.

After deletion, `/qr/{qr_code_id}` must no longer resolve to an active profile.

---

## AI Analysis Sessions

**Treatment:** Hard Delete.

All Analysis Session records owned by the account are permanently deleted, regardless of status (completed or failed).

There is no separate "accepted AI decision" entity in V1. Analysis Sessions are stored reports only; an accepted suggestion already exists as ordinary Product Domain data owned by the responsible domain (e.g. an accepted bio is already part of Profile Content, deleted in that step). Analysis Sessions are owned by the AI Platform and scoped to the account. They are deleted as part of the account data cascade.

---

## Binary Assets

**Treatment:** Physical deletion via Storage Platform.

All binary assets owned by the account are physically deleted from Storage:

* Avatar image
* Background image
* Uploaded images
* Generated QR assets

The Data Platform initiates deletion by removing asset references. The Storage Platform detects orphaned assets and executes physical deletion per its standard orphan cleanup process.

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

| Entity | Treatment | Owner of Decision |
|--------|-----------|-------------------|
| Sessions | Invalidated immediately | Authentication System |
| Out Links | Hard Delete | Data Platform |
| Analytics Events | Hard Delete | Data Platform |
| Profile Content | Hard Delete | Data Platform |
| Blocks | Hard Delete | Data Platform |
| Appearance State | Hard Delete | Data Platform |
| Connected Accounts | Hard Delete | Data Platform |
| QR Code Records | Hard Delete | Data Platform |
| AI Analysis Sessions | Hard Delete | AI Platform |
| Binary Assets | Physical deletion | Storage Platform |
| Account Record | Status → deleted | Data Platform |
| Username | Permanently reserved | Data Platform |
| Audit Logs | Retained 12 months | Audit Logging Policy |

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
