# qr-code.system.specification.v1.md

# Minime QR Code System Specification V1

**Version:** 1.2
**Status:** Approved
**Domain:** Account

> The QR Code System is an implementation area within the Account domain. QR Code is not a standalone Product Domain and must never become one.

---

# 1. Purpose

The QR Code System provides a permanent, scannable entry point to an Account's public profile.

Its only purpose is to let a visitor scan a code and arrive at the Account's current public profile route.

The QR Code System provides an alternative access method only. It must never alter navigation, rendering, analytics, or profile behavior.

---

# 2. Scope

The QR Code System is responsible only for:

* Generating exactly one QR Code per Account, mandatorily, before the Account becomes active
* Persisting the QR Code record as Account-owned data
* Resolving the QR Code's route to the Account's current public profile by reading `username` live from the Account record
* Providing the QR asset for download and share

The QR Code System must never introduce QR analytics, QR scan tracking, QR customization, or external QR providers.

---

# 3. Ownership

The QR Code System is owned entirely by the Account domain.

QR Code data:

* Must belong to the Account domain.
* Must remain an implementation area within Account — never a standalone Product Domain.
* Must never be owned, partially owned, or co-owned by any other domain.

The QR Code System never owns:

* Public Profiles
* Profile Content
* Rendering
* Analytics
* Out Links
* Username resolution

---

# 4. One Account, One QR Code

```text
One Account = One QR Code Record
One QR Code Record = One Account
```

Rules:

* Every Account owns exactly one QR Code record.
* The QR Code record must be created only once, as a mandatory step before the Account becomes active (see Section 8).
* A QR Code record must never be created more than once for the same Account.
* The QR Code record must be keyed by `account_id`.
* The QR Code record must never be keyed by `username`.
* `qr_code_id` must be globally unique. Implementation must enforce `PRIMARY KEY(qr_code_id)` (or `UNIQUE(qr_code_id)` if the primary key uses a separate surrogate column).
* `account_id` must be unique among QR Code records. Implementation must enforce `UNIQUE(account_id)`.
* No two QR Code records may reference the same Account.
* No two Accounts may share the same QR Code record.

---

# 5. Identity and Immutability

* The QR Code's identity is `qr_code_id`. It is globally unique, permanent, and immutable once created.
* `account_id` is required, unique, references the owning Account, and is immutable after creation.
* The QR Code must never be editable by the user.
* The user must never regenerate, replace, reset, customize, or delete the QR Code independently.
* The QR Code record must never store `username`, a username snapshot, or any duplicated public profile route data derived from username. Username is read live from the Account record through `account_id` whenever the QR Code's destination is resolved.
* Username changes are not supported in V1; the QR destination therefore remains stable in V1 by construction, not by caching a username value.

---

# 6. QR Payload and Routing

* The QR Code payload must be a URL only.
* The QR Code payload must never contain JSON, account data, profile data, authentication data, or embedded user information.
* The QR Code payload must always use this route format:

```text
https://minime.ae/qr/{qr_code_id}
```

* `/qr/{qr_code_id}` must resolve the QR Code record.
* `/qr/{qr_code_id}` must resolve `account_id` from the QR Code record.
* `/qr/{qr_code_id}` must resolve the Account's current `username` by reading the Account record through `account_id` — never from a stored value on the QR Code record.
* `/qr/{qr_code_id}` must redirect to `/{username}`.
* `/qr/{qr_code_id}` must never render a separate public profile page.
* `/qr/{qr_code_id}` must never expose internal Account data.
* If the QR Code record does not exist, belongs to a deleted Account, or cannot resolve to an active public profile, the route must fail safely (see `public-profile.routing.specification.v1.md` and `public-profile.error.states.v1.md`).

The canonical QR redirect flow is:

```text
qr_code_id
↓
AccountQRCode
↓
account_id
↓
Account.username
↓
Redirect to /{username}
```

The full routing contract is defined in `Architecture-Product-Definition/public-profile/public-profile.routing.specification.v1.md`.

---

# 7. Generation

QR Code generation is handled internally by Minime only.

* Minime must never introduce an external QR provider.
* Minime must never introduce provider abstraction for QR generation.
* QR generation must use an internal QR library or equivalent implementation dependency.
* QR generation must have no external API dependency, no vendor lock, no per-generation provider cost, and no rate-limit dependency.

The full generation contract is defined in `qr-code.generation.specification.v1.md`.

---

# 8. Mandatory Creation Before Account Activation

```text
An Account cannot reach active status unless its AccountQRCode record and canonical SVG asset have been successfully created and persisted.
```

Rules:

* QR Code creation is mandatory for every active Account. There is no active Account without a QR Code record.
* QR Code creation must occur during the account creation transaction, or as a required blocking post-creation step that must succeed before the Account becomes active.
* If QR Code record creation fails, the Account must not become active.
* If canonical SVG asset generation or persistence fails, the Account must not become active.
* The system must never allow an active Account without a QR Code record.
* The system must never allow an active Account whose QR Code record references a missing canonical SVG asset.
* Lazy QR generation — on first dashboard view, first settings view, first download, first share, or first public profile visit — is forbidden.

The full creation-flow contract is defined in `qr-code.generation.specification.v1.md`.

---

# 9. Asset Format

* The canonical QR Code asset format is SVG.
* PNG is supported only as an export/download format.
* PNG export/download must never create a second QR Code record.
* SVG remains the canonical stored asset in all cases.

---

# 10. Persistence

* The QR Code record must be persisted as Account-owned data.
* The generated SVG asset must be stored through the Storage Platform.
* The QR asset must be classified as a generated asset, never a user-uploaded asset.

---

# 11. Account Settings Presentation

* The QR Code must be available only inside `Settings / Account / QR Code`.
* The QR Code must be visible directly in that section. A separate `View` action must never be required.
* The only supported user actions are:

```text
Download
Share
Copy profile link
```

* The following actions must never be exposed: `View`, `Print`, `Customize`, `Regenerate`, `Delete`, `Edit destination`.
* Download and Share must read the existing QR Code record and asset. Download and Share must never generate a new QR Code record and must never modify the QR Code record.

---

# 12. Relationship with Public Profile

Each QR Code references exactly one Account and that Account's current public profile route.

The public profile route remains the destination. The QR Code is only an alternative access mechanism and must never modify the public profile.

---

# 13. Relationship with Rendering

The QR Code System is independent from Rendering.

Rendering produces the public profile. The QR route only redirects to the resulting public profile URL. The QR Code System must never participate in the rendering pipeline.

---

# 14. Relationship with Out Links

QR Codes resolve directly to the public profile route. They must never participate in the Out Links system. Once the public profile is loaded, any outbound navigation continues to follow the Out Links architecture unchanged.

---

# 15. Regeneration Policy

* User-triggered regeneration is forbidden.
* System-triggered regeneration is allowed only for: missing asset recovery, corrupted asset recovery, storage migration, format migration.
* System regeneration must preserve the same QR Code record, the same `qr_code_id`, the same `account_id`, the same logical QR identity, and the same QR payload route.
* System regeneration must never create a new QR Code record.

---

# 16. Account Deletion

* Account deletion must delete the QR Code record.
* Account deletion must delete the generated QR asset through the existing Storage Platform deletion/orphan cleanup rules.
* After account deletion, `/qr/{qr_code_id}` must no longer resolve to an active profile.
* This must remain aligned with the existing account deletion cascade defined in `account.deletion.policy.v1.md`.

---

# 17. Invariants

The following architectural rules are permanent.

## Account Ownership

QR Code must always belong to the Account domain. It must never become a standalone Product Domain.

---

## One Account, One Record — Both Directions

`UNIQUE(account_id)` always holds: no Account owns more than one QR Code record. `PRIMARY KEY(qr_code_id)` (or `UNIQUE(qr_code_id)`) always holds: no QR Code record is shared by more than one Account.

---

## Permanent Identity

`qr_code_id` is globally unique, permanent, immutable, and never reused.

---

## Username Is Not Identity, Never Stored

The QR Code must never be keyed, identified, or addressed by `username`. The QR Code record must never store `username` or a username snapshot. Username is always resolved live from the Account record through `account_id`.

---

## URL-Only Payload

The QR payload must always be a URL and must never embed structured data.

---

## No External Provider

QR generation must always be internal to Minime.

---

## No User Regeneration

Only the system may regenerate a QR asset, and only for recovery or migration reasons. The user must never trigger regeneration.

---

## No Customization

QR Code must never expose styling, color, or design customization to the user.

---

## No Analytics or Tracking

QR Code must never own scan analytics or scan tracking in V1.

---

## SVG Canonical

The stored canonical QR asset format is always SVG. PNG exists only as an export/download format.

---

## Mandatory Before Activation

An Account must never reach active status without a successfully created and persisted QR Code record and canonical SVG asset. Lazy generation at any later trigger point is forbidden.

---

# 18. Out of Scope (V1)

The following are explicitly out of scope for V1 and must not be implemented:

* QR scan analytics
* QR scan tracking
* QR customization (color, style, logo, branding)
* External QR generation providers
* Multiple QR Codes per Account
* User-triggered regeneration, replacement, reset, or deletion
* `View`, `Print`, `Customize`, `Regenerate`, `Delete`, and `Edit destination` actions
* Storing `username` or a username snapshot on the QR Code record
* Lazy QR generation triggered by any user-facing view or action

This specification is the canonical QR Code architecture for Minime V1.
