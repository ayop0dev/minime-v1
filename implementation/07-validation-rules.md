# Minime V1 — Validation Rules

## Status

Canonical. Final.

---

## Architecture Authority

```
Architecture-Product-Definition/account/authentication.policy.v1.md
Architecture-Product-Definition/account/account.model.specification.v1.md
Architecture-Product-Definition/blocks/block.system.specification.v1.md
Architecture-Product-Definition/blocks/button.block.specification.v1.md
Architecture-Product-Definition/blocks/avatar.block.specification.v1.md
Architecture-Product-Definition/blocks/name.block.specification.v1.md
Architecture-Product-Definition/blocks/bio.block.specification.v1.md
Architecture-Product-Definition/blocks/image.block.specification.v1.md
Architecture-Product-Definition/blocks/social-icons.block.specification.v1.md
Architecture-Product-Definition/blocks/divider.block.specification.v1.md
Architecture-Product-Definition/blocks/title.block.specification.v1.md
Architecture-Product-Definition/blocks/textbox.block.specification.v1.md
Architecture-Product-Definition/out-links/out-link.system.specification.v1.md
Architecture-Product-Definition/analytics/analytics.model.specification.v1.md
implementation/03-canonical-data-model.md
```

---

## Purpose

Defines canonical validation rules for Minime V1.

Validation rules define:

- validation ownership
- validation execution order
- validation layer boundaries
- concrete validation constants derived from frozen architecture
- per-domain field schemas

The backend is the final validation authority. Frontend validation exists for user experience only. Client validation must never replace backend validation.

---

## Validation Layers

Validation executes in this exact order:

```
HTTP Request
    │
    ▼
Transport Validation
    │
    ▼
Schema Validation
    │
    ▼
Authentication
    │
    ▼
Authorization
    │
    ▼
Business Validation
    │
    ▼
Persistence Validation
    │
    ▼
Business Execution
```

No implementation may bypass any required validation layer.

---

## Transport Validation

Verifies request structure before schema inspection.

Must verify:

- Content-Type matches expected type
- Request size is within limits
- Encoding is valid (UTF-8)
- Required headers are present

Invalid transport requests terminate immediately.

---

## Schema Validation

Verifies field presence, types, and format.

Must verify:

- Required fields exist and are non-null
- Field types match expected types
- Field lengths are within bounds
- Numeric values are within ranges
- Enum values belong to approved sets
- Object structure matches schema
- Array structure and element schema

Schema validation must not execute business rules.

---

## Authentication Validation

Must verify:

- Bearer access token presence
- Token format (JWT)
- Token expiry
- Token signature integrity

Authentication must complete before authorization executes.

---

## Authorization Validation

Must verify:

- The authenticated account_id matches the resource's account_id
- No resource crosses account boundaries

Authorization must execute before business logic.

---

## Business Validation

Verifies Product Domain constraints that require database reads.

May verify:

- Username availability
- Block instance limits (identity blocks: max 1 per account)
- Reservation existence and expiry
- OutLink ownership before archiving

Business validation belongs to the owning Product Domain.

There is no "publication rule" validation. Minime V1 has no publishing workflow.

---

## Persistence Validation

Verifies database consistency before write.

May verify:

- Uniqueness constraints (username, auth identity, public_id)
- Referential integrity (account_id FK)
- Transaction prerequisites

---

## Domain Validation Constants

The following constants are derived directly from frozen architecture specifications. They must not be changed without updating the corresponding architecture document.

---

### Authentication

**Supported authentication methods:** Google Sign-In and Apple Sign-In only. Email OTP, Phone, SMS, WhatsApp, Password, Facebook, X, and LinkedIn authentication are not supported in V1.

**Route note:** Every schema below applies to a provider-agnostic route. `provider` is always a request-body field, validated against the V1 allowlist, never a path parameter.

**OAuth start schema (`POST /auth/oauth/start`):**

```
provider: body field, must equal "google" or "apple"; any other value rejected with 400 before any Provider Adapter is resolved
intent: body field, must equal "authenticate" (default) or "link"; "link" requires an authenticated session
```

**OAuth callback schema (`POST /auth/oauth/callback`):**

```
provider: body field, must equal "google" or "apple"; must match the provider bound to the state context recorded at startOAuth time — mismatch is rejected
code: string, non-empty, required
state: string, non-empty, required; must match a persisted, unexpired, unconsumed state value
```

The callback handler additionally validates, server-side, inside the resolved Provider Adapter, before extracting any identity data: `id_token` signature (against the provider's published keys), `iss` (issuer), `aud` (audience), `exp` (expiry), and `nonce` (must match the value persisted at startOAuth time). A token failing any one of these checks must be rejected; no identity data is extracted.

**Registration completion schema (`POST /auth/oauth/register`):**

```
provider: body field, must equal "google" or "apple"; must match the provider embedded in oauth_handoff_token — mismatch is rejected
oauth_handoff_token: string, non-empty, single-use, short-lived
reservation_id: UUID, required
```

**Provider linking finalization schema (`POST /account/auth-providers/link`):**

```
link_handoff_token: string, non-empty, single-use, short-lived; its bound account_id must match the authenticated caller — mismatch is rejected
```

**Normalized provider identity fields, produced only by `AuthenticationProviderAdapter.normalizeIdentity` (never accepted as raw client input on any route):**

```
provider_subject: string, non-empty — the verified sub claim; the only authentication identity key
email: string, optional, valid email format if present — never validated as a uniqueness constraint, never used for identity matching
email_verified: boolean, optional
provider_profile: object, optional — generic non-authoritative metadata (e.g. display_name, avatar_url, locale); not validated beyond being a JSON object; never interpreted for authorization
```

**Provider unlinking validation (`DELETE /account/auth-providers/{auth_identity_id}`):**

```
auth_identity_id: path parameter, UUID, required; must belong to the authenticated account
```

Unlinking must be rejected (`409`) if removing this identity would leave the account with zero `AuthenticationIdentity` records.

---

### Username

**From `account.model.specification.v1.md` and `03-canonical-data-model.md`:**

| Rule | Value |
|---|---|
| Allowed characters | `[a-z0-9]` only |
| Minimum length | 3 characters |
| Maximum length | 30 characters |
| Case | Lowercase only; uppercase input must be normalized to lowercase |
| Mutability | Immutable after registration. No update path in V1. |

**Username schema:**

```
username: string, pattern /^[a-z0-9]{3,30}$/, after lowercase normalization
```

Uniqueness check must consult both `Account.username` (permanent) and active `UsernameReservation` records.

---

### Profile Content

**From `profile.content.specification.v1.md` via `04-service-contracts.md`:**

| Field | Rule |
|---|---|
| `display_name` | string, required, non-empty |
| `bio` | string, nullable, max 1000 characters |
| `contact.email` | string, nullable, valid email format |
| `contact.phone` | string, nullable |
| `contact.whatsapp` | string, nullable |
| `contact.location` | string, nullable |
| `appearance_config.selected_theme_id` | string, nullable; if non-null must be a valid ID from the theme catalog at `packages/config/themes.ts` |

---

### Account Settings

**From `03-canonical-data-model.md` and `12-seo-and-integrations.md`:**

`Account.settings` is a JSONB field. V1 supports exactly one named field:

| Field | Rule |
|---|---|
| `gtm_container_id` | string, nullable; if non-null must be a non-empty string matching `/^GTM-[A-Z0-9]+$/`; max 20 characters |

No other fields are permitted in `Account.settings` in V1. Unknown fields must be rejected at the schema validation layer.

---

### Block Validation

**From individual block specifications:**

All blocks must satisfy:

```
type: BlockType enum value (required)
content: object matching per-type schema (required)
settings: object matching per-type schema (required)
sort_order: integer, >= 0 (required)
```

**Per-type content schemas:**

| BlockType | `content` schema |
|---|---|
| `avatar` | `{}` — empty object only |
| `name` | `{}` — empty object only |
| `bio` | `{}` — empty object only |
| `image` | `{ image_id: string (required, UUID, must reference an ImageAsset owned by the same account) }` |
| `social_icons` | `{ accounts: Array<{ connected_account_id: string (UUID), sort_order: integer }>  }` |
| `button` | `{ label: string (required, non-empty), url: string (required, valid URL) }` |
| `divider` | `{}` — empty object only |
| `title` | `{ text: string (required, non-empty) }` |
| `textbox` | `{ text: string (required, non-empty) }` |

**Per-type settings schemas:**

| BlockType | `settings` schema |
|---|---|
| `avatar` | `{ source: "profile_avatar" }` — literal value, no other values accepted |
| `name` | `{ source: "profile_display_name" }` — literal value, no other values accepted |
| `bio` | `{ source: "profile_bio" }` — literal value, no other values accepted |
| `image` | `{}` — empty object only |
| `social_icons` | `{}` — empty object only |
| `button` | `{}` — empty object only |
| `divider` | `{}` — empty object only |
| `title` | `{}` — empty object only |
| `textbox` | `{}` — empty object only |

**Button settings note:** `open_in_new_tab` is not a V1 field. ButtonBlockSettings = {} per architecture.

**Social icons settings note:** `show_labels` is not a V1 field. SocialIconsBlockSettings = {} per architecture.

**Divider content note:** `has_label` and `label` are not V1 fields. Divider V1 is "Visual Only" — no content.

**Identity block instance limit:**

The system must reject creation of a second `avatar`, `name`, or `bio` block for an account that already has one of that type.

**Total block limit (MAX_BLOCKS_PER_ACCOUNT):**

The system must reject block creation when total active (non-soft-deleted) block count for the account is at or above MAX_BLOCKS_PER_ACCOUNT (canonical definition and implementation default in `03-canonical-data-model.md`). This is a V1 operational safety limit. Do not introduce plan tiers.

**social_icons connected_account_id validation:**

Each `connected_account_id` in a social_icons block's `content.accounts` array must reference an existing `ConnectedAccount` owned by the same account (business validation layer). Validation must reject additions of unknown or cross-account references. When a connected_account_id is removed from block content via ConnectedAccountsService.removeConnectedAccount, no dangling reference may remain.

**social_icons internal `content.accounts[].sort_order` validation:** this is a nested value inside a single Block's JSONB `content`, not a separate table, so it cannot be a database constraint — it is enforced entirely at this validation layer, on every write to a social_icons block's `content`: values must be unique within the array (no two entries share a `sort_order`), contiguous starting at 0 (`0..N-1` for `N` entries, no gaps), and cover exactly the entries present in the array (a write with a non-contiguous, duplicated, or out-of-range set of `sort_order` values is rejected, not auto-corrected).

**`style_overrides` field validation:**

`style_overrides` is an optional JSONB field on Block. Validation rules:

| Rule | Requirement |
|---|---|
| Type | Object (key-value map) or null |
| Null | Accepted — means the block inherits all styles from the active theme |
| Key format | String keys only; no nested objects deeper than one level per key |
| Forbidden values | Resolved styles and inherited theme values must not be stored here; the system must reject any write that contains a resolved style blob |
| Allowed keys | Closed per block type — see `Architecture-Product-Definition/block-styling/block-style.model.specification.v1.md` — "Closed Property Catalog (V1)". A key not in that block type's list is rejected (the write fails; it is never silently dropped or ignored). |
| Allowed values | Each key's value must be one of the domain values listed for it in the same Closed Property Catalog, and must additionally satisfy the active Theme's constraint range for that key (`theme.constraints.specification.v1.md`) — see `block-style.constraints.specification.v1.md` for the Reject-Save (write time) vs Reset-To-Inherited (theme-switch time) split. |

Reference Blocks (`avatar`, `name`, `bio`) do have a closed set of user-controllable style keys — see the Closed Property Catalog. They have no user-controllable *content* (content is sourced from `ProfileContent`), which is a separate concern from styling.

**Block type enum:** See `03-canonical-data-model.md` — BlockType enum (9 values).

---

### Connected Accounts

| Field | Rule |
|---|---|
| `platform` | SocialPlatform enum value (required) |
| `username` | string, non-empty, format validated per platform-specific rules |

Platform-specific username validation rules live in:

```
Architecture-Product-Definition/social-accounts/social-platforms/
```

**SocialPlatform enum:** See `03-canonical-data-model.md` — SocialPlatform enum (12 values).

`url` must never be accepted from the client. URL is always system-generated by SocialAccountsService.

---

### Out Links

**public_id format (from `out-link.system.specification.v1.md`):**

```
8 characters, pattern /^[a-z0-9]{8}$/, non-sequential, system-generated
```

`public_id` is never accepted from the client. It is generated by OutLinksService at creation time.

---

### Analytics

**AnalyticsEventType enum:** See `03-canonical-data-model.md` — AnalyticsEventType enum (2 values). `out_link_click` and `qr_scan` are not valid types.

**DeviceCategory enum:** See `03-canonical-data-model.md` — DeviceCategory enum (3 values). `unknown` is not a valid device category.

---

### Account Deletion

Deletion request must include:

```
confirmation: boolean, must equal true
```

---

## AI Platform Output Validation

AI responses must be treated as untrusted input and validated before use. Validation executes inside the AI Platform service layer, before any accepted content is passed to a Product Domain service.

Rules:

| Rule | Requirement |
|---|---|
| Schema version conformance | AI response must conform to the configured current `output_schema_version`. If the response does not conform, the Analysis Session is marked `failed`; an empty suggestion array is returned; no error is surfaced to the user. |
| JSON format | AI response must be valid JSON. If parsing fails, the response is discarded, the Analysis Session is marked `failed`, and an empty suggestion array is returned; no error is surfaced to the user. |
| Maximum text length | Suggested text values must not exceed the corresponding field's canonical maximum (e.g. bio suggestions must not exceed 1000 characters per `07-validation-rules.md` — Profile Content). |
| No HTML or script content | Suggested text values must be treated as plain text. HTML tags and script content must be stripped or rejected before acceptance. |
| Unknown fields | Unexpected fields in AI response objects are ignored; they must never be passed to Product Domain services or persisted. |
| Suggestion count | AI Platform must not return more suggestions than the configured maximum per request (implementation-configured constant). Excess suggestions are truncated. |

`output_schema_version` is a Minime-owned semantic version string, never provider- or model-owned. AI output validation belongs to the AI Platform service layer. Product Domain services must not receive raw AI output.

**Version string validation:** `analysis_version` and `output_schema_version` must each be a valid semantic version string in `MAJOR.MINOR.PATCH` format (e.g. `"1.0.0"`). Numeric version values are not valid. `provider_key` and `model_key`, where present, are diagnostic metadata only and are not subject to AI output validation rules — they must never be used to determine validity or reuse.

---

## URL Validation

URLs submitted via Button Block must satisfy:

- Absolute URL (not relative)
- Approved schemes: `http`, `https`, `mailto`, `tel`
- Valid host when scheme requires it

Unknown or disallowed URL schemes must be rejected.

---

## File Upload Validation

Avatar uploads:

- Content-Type: `multipart/form-data`
- Accepted MIME types: `image/jpeg`, `image/png`, `image/webp`, `image/heic` — matches `storage.architecture.specification.v1.md` — "Canonical Image Format" exactly. `image/gif` is never accepted (animated GIF is an explicitly unsupported media type — see "Unsupported Media Types" in the same document); GIF support is not a partial exception for static GIFs either — the MIME type is rejected outright, regardless of whether the specific file is animated.
- Magic-byte content sniffing must confirm the file's actual content matches its declared Content-Type before processing; a mismatched or malformed file is rejected before reaching the image-processing step (see `platform/storage/asset.processing.specification.v1.md` — "Input Validation").
- Maximum file size: implementation-configured
- Executable file types must be rejected

Image block uploads (`POST /api/v1/profile/images`):

- Content-Type: `multipart/form-data`
- Accepted MIME types: `image/jpeg`, `image/png`, `image/webp`, `image/heic` — same rule as avatar uploads above
- Maximum file size: implementation-configured (same limit as avatar)
- Executable file types must be rejected unconditionally
- `image_id` in block content must be a valid UUID referencing an `ImageAsset` owned by the same account (business validation layer)

---

## ImageAsset Validation

| Rule | Value |
|---|---|
| `image_id` in block content | Required, UUID format, must reference an existing `ImageAsset` with matching `account_id` |
| Accepted upload MIME types | `image/jpeg`, `image/png`, `image/webp`, `image/heic` (never `image/gif`) |
| Maximum file size | Implementation-configured |
| Executable file types | Must be rejected unconditionally |
| Ownership on delete | `ImageAsset.account_id` must match the authenticated `account_id` |
| Reference check on delete | No active (non-soft-deleted) block may reference the `image_id` being deleted |

ImageAsset validation belongs to the Profile domain. Post-upload processing (decode, transcode to WebP, dimension/decompression-bomb limits) is Storage Platform's responsibility — see `platform/storage/asset.processing.specification.v1.md`.

---

## Enum Validation Rule

Unknown enum values must always be rejected. Enum validation executes at schema validation layer, before business logic.

Full enum definitions are in `03-canonical-data-model.md`.

---

## Validation Implementation

- Validation schemas use **Zod**.
- Shared schemas live in `packages/validation`.
- Backend validation is authoritative; frontend reuses schemas for UX only.
- Backend never trusts frontend validation results.

---

## Validation Prohibitions

Implementation must not:

- trust client validation
- skip schema validation
- execute business logic before validation completes
- persist invalid data
- publish events before validation completes
- bypass Product Domain ownership
- validate `profile_id` — it does not exist
- validate `password` fields — V1 has no password authentication
- validate `channel` on any OTP endpoint — Email OTP does not exist in V1, and no OTP endpoint exists to validate against
- accept `provider_subject` as a client-provided field anywhere — it is always derived from a verified `id_token` during an OAuth callback
- accept a `provider` value other than `google` or `apple` on any authentication or account-provider endpoint in V1
- treat `provider` as a route/path segment on any authentication endpoint — it is always request-body data
- accept a client-supplied `client_secret`, Apple private key, or any provider secret on any endpoint — these exist only inside the backend's server-to-server token exchange, owned by the relevant Provider Adapter
- accept `email` as a field used to locate, match, or merge an existing Account on any endpoint — identity resolution is always `(provider, provider_subject)`
- accept a top-level `display_name` field for `AuthenticationIdentity` on any endpoint — provider-reported names live only inside `provider_profile`
- validate `open_in_new_tab` in button settings — not a V1 field
- validate `show_labels` in social_icons settings — not a V1 field
- validate `has_label` or `label` in divider content — not a V1 field
- accept `url` as a client-provided field for ConnectedAccount — URL is system-generated
- accept `public_id` as a client-provided field for OutLink — public_id is system-generated
- accept `storage_key` as a client-provided field for ImageAsset — storage key is system-generated and must not be exposed or accepted from clients
- accept cross-account `image_id` references — ImageAsset.account_id must match the block's account_id
