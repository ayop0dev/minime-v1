# qr-code.lifecycle.specification.v1.md

# Minime QR Code Lifecycle Specification V1

**Version:** 1.2
**Status:** Approved
**Domain:** Account

---

# 1. Purpose

This specification defines the lifecycle of the QR Code record within Minime.

It describes how the QR Code is created, maintained, and retired throughout the lifecycle of its owning Account.

---

# 2. Scope

The QR Code Lifecycle defines:

* Creation
* Availability
* System-triggered regeneration
* Retirement (account deletion)

It does not define QR generation mechanics (`qr-code.generation.specification.v1.md`) or routing behavior (`public-profile.routing.specification.v1.md`).

---

# 3. Lifecycle Ownership

The Account domain owns the complete lifecycle of the QR Code record.

The QR Code lifecycle always follows the lifecycle of its owning Account. A QR Code record must never exist without an owning Account.

---

# 4. Lifecycle States

```text
Not Created

↓

Generated (once, mandatory, before the Account becomes active)

↓

Account Activated (blocked until QR Code record + SVG asset exist)

↓

Available

↓

System Regenerated (only for recovery or migration)

↓

Retired (on Account deletion)
```

There is no user-triggered transition anywhere in this lifecycle. There is no path from "Not Created" directly to "Account Activated" — generation must complete first.

---

# 5. Creation

* The QR Code record becomes eligible for creation only once the Account and its canonical public profile route exist.
* Creation happens exactly once per Account.
* Creation establishes the permanent association between the Account and its QR Code: `account_id`, `qr_code_id`, and `qr_url`.
* A QR Code record must never be created a second time for the same Account.
* Creation must complete successfully — record persisted and canonical SVG asset persisted — before the Account becomes active. If creation fails, the Account must not become active.
* The QR Code record never stores `username` or a username snapshot; only `account_id` is stored as the reference to the owning Account.

---

# 6. Availability

After successful creation, the QR Code is available for use.

At this stage it may be:

* Viewed directly inside `Settings / Account / QR Code`
* Downloaded
* Shared
* Used to copy the profile link

Availability never modifies the QR Code record or its asset.

---

# 7. System-Triggered Regeneration

Regeneration of the underlying asset (not the record) may occur only for:

* Missing asset recovery
* Corrupted asset recovery
* Storage migration
* Format migration

Regeneration must always preserve:

* The same QR Code record
* The same `qr_code_id`
* The same `account_id`
* The same logical QR identity
* The same QR payload route (`qr_url`)

Regeneration must never create a new QR Code record and must never be triggered by the user.

---

# 8. Retirement

The QR Code record is retired only when its owning Account is deleted.

Retirement:

* Deletes the QR Code record.
* Deletes the generated SVG asset through the Storage Platform deletion/orphan cleanup rules.
* Causes `/qr/{qr_code_id}` to no longer resolve to an active profile.

A retired QR Code must never return to an active state. This must remain aligned with the existing account deletion cascade (`account.deletion.policy.v1.md`).

---

# 9. Relationship with Account Lifecycle

```text
Account Created (pending activation)

↓

Canonical Public Profile Route Established

↓

QR Code Created (record + canonical SVG asset persisted)

↓

Account Activated

↓

Account Deleted

↓

QR Code Retired
```

The QR Code lifecycle never outlives the Account lifecycle, and it never begins before the Account and its canonical route exist. The Account never reaches "Activated" before "QR Code Created" completes.

---

# 10. Relationship with Public Profile

The Account's current public profile route determines the resolution target of `/qr/{qr_code_id}` at request time, resolved live through `account_id → Account.username` — never from a value stored on the QR Code record.

Changes to Profile Content or Appearance never require a new QR Code record, because the QR payload references `qr_code_id`, not profile content, and the username is not supported as editable in V1.

---

# 11. Lifecycle Rules

* Every active Account has exactly one QR Code record, enforced by `UNIQUE(account_id)`.
* Every QR Code record belongs to exactly one Account, enforced by `PRIMARY KEY(qr_code_id)` (or `UNIQUE(qr_code_id)`).
* A QR Code record never exists without an owning Account.
* An Account never reaches active status without a QR Code record and a persisted canonical SVG asset.
* A retired QR Code record cannot return to an active state.
* System-triggered regeneration always preserves logical identity (`qr_code_id`, `account_id`, `qr_url`).
* User-triggered regeneration, replacement, reset, customization, or deletion is forbidden at every stage of the lifecycle.
* The QR Code record never stores `username` or a username snapshot at any stage of the lifecycle.

---

# 12. Invariants

The following architectural rules are permanent.

## Account Dependency

The QR Code lifecycle is fully dependent on the Account lifecycle.

---

## Single Record, Both Directions

Only one QR Code record exists per Account (`UNIQUE(account_id)`), for the entire lifetime of that Account. Only one Account exists per QR Code record (`PRIMARY KEY(qr_code_id)` / `UNIQUE(qr_code_id)`).

---

## Permanent Identity

`qr_code_id` and `account_id` never change across any lifecycle transition.

---

## No Stored Username

The QR Code record never stores `username` or a username snapshot at any lifecycle stage. Resolution always reads `Account.username` live through `account_id`.

---

## Mandatory Before Activation

No Account may reach active status without a successfully created and persisted QR Code record and canonical SVG asset. There is no lazy-generation fallback at any later trigger point.

---

## Asset May Be Regenerated, Record Never Recreated

The stored SVG asset may be system-regenerated for recovery or migration. The QR Code record itself is created exactly once and is never recreated.

---

## Deletion-Only Retirement

The only path to retirement is Account deletion.

---

## No User-Triggered Transitions

No lifecycle state transition may be initiated by the user.

---

# 13. Out of Scope (V1)

The following are explicitly out of scope and must not be implemented:

* Versioned QR assets
* Multiple presentation formats stored as canonical
* QR expiration policies
* Campaign-specific lifecycle policies
* Any lifecycle transition triggered by the user
* Storing `username` or a username snapshot on the QR Code record
* Lazy generation triggered after Account activation

This lifecycle is the canonical lifecycle for QR Codes within Minime V1.
