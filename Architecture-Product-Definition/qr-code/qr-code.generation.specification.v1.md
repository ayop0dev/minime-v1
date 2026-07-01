# qr-code.generation.specification.v1.md

# Minime QR Code Generation Specification V1

**Version:** 1.2
**Status:** Approved
**Domain:** Account

---

# 1. Purpose

This specification defines the canonical generation model for QR Codes within Minime.

It establishes when QR Codes are generated, what they represent, and the architectural rules governing their creation. It does not redefine ownership, persistence, or routing — those are defined in `qr-code.system.specification.v1.md`.

---

# 2. Scope

QR Code generation is responsible only for:

* Producing the canonical SVG QR asset for an Account
* Encoding the Account's canonical `/qr/{qr_code_id}` route as the QR payload
* Saving the generated asset through the Storage Platform
* Saving the QR Code record under Account-owned data

QR Code generation is never responsible for lifecycle management beyond creation and system-triggered regeneration, rendering, analytics, or profile routing decisions.

---

# 3. Generation Trigger

QR Code generation becomes eligible only after:

* The Account exists.
* The Account's canonical public profile route exists.

The QR Code must be created once, at that point. It must never be generated for an incomplete, temporary, or unestablished Account.

QR generation must never be deferred to a user-facing request (e.g. it must never be generated lazily on first dashboard view, first settings view, first download, first share, or first public profile visit). It is created as part of Account creation, before the Account becomes active.

---

# 4. Mandatory Creation Before Account Activation

```text
An Account cannot reach active status unless its AccountQRCode record and canonical SVG asset have been successfully created and persisted.
```

Rules:

* QR Code creation is mandatory for every active Account.
* QR Code creation must occur during the account creation transaction, or as a required blocking post-creation step before the Account becomes active.
* If QR Code record creation fails, the Account must not become active.
* If canonical SVG asset generation or persistence fails, the Account must not become active.
* The system must never allow an active Account without a QR Code record.
* The system must never allow an active Account whose QR Code record references a missing canonical SVG asset.
* Lazy generation at any later trigger point is forbidden — there is no fallback path that creates the QR Code after the Account is already active.

---

# 5. Generation Flow

```text
Account created

↓

Canonical QR route created

↓

QRService.generate(account_id, qr_code_id, qr_url)

↓

SVG QR asset generated

↓

Asset saved through Storage Platform

↓

QR Code record saved under Account-owned data

↓

Account becomes active
```

This is the only generation flow in V1. Account activation never precedes successful completion of every step above.

---

# 6. Generation Target

The generated QR Code always encodes:

```text
https://minime.ae/qr/{qr_code_id}
```

It never encodes:

* The raw `/{username}` profile URL directly
* Preview URLs
* Editor URLs
* Temporary URLs
* Internal system endpoints
* JSON, account data, profile data, authentication data, or embedded user information

The `qr_code_id` route is permanent and resolves to the Account's current public profile at scan time, by reading `username` live from the Account record through `account_id`. This indirection is what keeps the QR payload stable even though `/qr/{qr_code_id}` resolves to `/{username}` at request time. The QR Code record never stores `username` or a username snapshot.

---

# 7. Canonical Identity

The QR Code represents the permanent identity of the Account's scannable entry point.

* `qr_code_id` is the QR Code's identity. It is globally unique (`PRIMARY KEY(qr_code_id)` or `UNIQUE(qr_code_id)`), immutable, and never reused.
* `account_id` is the owning Account reference. It is required, unique (`UNIQUE(account_id)`), and immutable after creation.
* The QR Code never defines the destination directly — it always resolves through `/qr/{qr_code_id}` to the Account's current public profile route.

---

# 8. Generation Rules

* Exactly one QR Code record exists per Account, enforced by `UNIQUE(account_id)`.
* Exactly one Account exists per QR Code record, enforced by `PRIMARY KEY(qr_code_id)` (or `UNIQUE(qr_code_id)`).
* Generation occurs exactly once per Account, before the Account becomes active.
* QR generation must never alter Account data, Profile Content, or any other Product Domain data.
* QR generation must be internal to Minime: no external provider, no provider abstraction, no external API dependency, no vendor lock, no per-generation provider cost, no rate-limit dependency.
* QR generation must use an internal QR library or equivalent implementation dependency to produce the SVG asset.

---

# 9. Regeneration Rules

System-triggered regeneration is allowed only for:

* Missing asset recovery
* Corrupted asset recovery
* Storage migration
* Format migration

User-triggered regeneration is forbidden in all cases.

Regeneration must always preserve the same QR Code record, the same `qr_code_id`, the same `account_id`, and the same `qr_url`. Regeneration must never create a new QR Code record.

---

# 10. Relationship with Public Profile Routing

The Account's canonical public profile route provides the resolution target for `/qr/{qr_code_id}`.

QR generation depends on the existence of the canonical public profile route. The public profile route never depends on QR generation. The dependency is one-directional.

---

# 11. Relationship with Rendering

Rendering produces the public profile. QR generation never participates in rendering and never renders the public profile itself.

---

# 12. Relationship with Analytics

QR generation performs no analytics and records no scan events. QR scan tracking and QR analytics are out of scope for V1 and must not be implemented.

---

# 13. Validation

Before generation, the system must verify:

* The Account exists.
* The Account's canonical public profile route exists.

Generation must fail safely if these conditions are not satisfied; it must never partially persist a QR Code record without its asset. If generation fails for any reason, the Account must not become active.

---

# 14. Generated Asset

The generated QR asset must:

* Be the canonical SVG representation of `https://minime.ae/qr/{qr_code_id}`.
* Be reusable for both digital display and download.
* Remain independent from any presentation styling.
* Exist and be persisted before the owning Account becomes active.

PNG, if needed for download compatibility, is a derived export format only. PNG generation must never create or modify a QR Code record, and must never replace SVG as the canonical stored asset.

---

# 15. Invariants

The following architectural rules are permanent.

## One QR Per Account, One Account Per QR

An Account owns exactly one QR Code record, enforced by `UNIQUE(account_id)`. A QR Code record belongs to exactly one Account, enforced by `PRIMARY KEY(qr_code_id)` (or `UNIQUE(qr_code_id)`).

---

## Canonical Route Payload

Every generated QR Code encodes `https://minime.ae/qr/{qr_code_id}` and nothing else.

---

## Immutable Identity

`qr_code_id` and `account_id` never change after creation.

---

## No Stored Username

The QR Code record never stores `username` or a username snapshot. Resolution always reads `Account.username` live through `account_id`.

---

## No Business Logic

QR generation contains no business rules beyond producing the SVG asset and persisting the record.

---

## No Rendering, No Routing Ownership

QR generation never renders the public profile and never defines URL routing behavior beyond its own `/qr/{qr_code_id}` resolution, which is owned by Routing/Public Profile.

---

## No Analytics

QR generation never records analytics events.

---

## Internal Generation Only

QR generation must always be performed internally by Minime, using an internal QR library or equivalent implementation dependency — never an external provider or provider abstraction.

---

## Deterministic Payload

Generating the QR Code for a given Account always encodes the same `qr_code_id` route for the lifetime of that Account.

---

## Mandatory Before Activation

No Account may become active without a successfully created and persisted QR Code record and canonical SVG asset. There is no lazy-generation fallback.

---

# 16. Out of Scope (V1)

This specification intentionally excludes:

* External QR generation providers
* QR customization (styling, color, logo, branding)
* QR scan analytics or tracking
* Multiple QR Codes per Account
* Batch generation
* User-triggered regeneration
* Storing `username` or a username snapshot on the QR Code record
* Lazy generation triggered by any user-facing view or action

This generation model is the canonical QR generation model for Minime V1.
