# Minime V1 — Canonical Data Model

## Purpose

This document defines the canonical persistent data model for Minime V1.

It specifies:

- every persistent entity and its fields
- every enum
- every relationship
- every uniqueness constraint
- every index
- Prisma model specifications
- persistence rules

This document is the single authoritative reference for all database schema work.

Every repository, service, migration, and Prisma model must conform to this document.

---

## Architecture Authority

This document implements the frozen architecture defined in:

```
Architecture-Product-Definition/docs/MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md
Architecture-Product-Definition/account/account.model.specification.v1.md
Architecture-Product-Definition/account-management/connected.accounts.specification.v1.md
Architecture-Product-Definition/profile-content/profile.content.specification.v1.md
Architecture-Product-Definition/blocks/block.system.specification.v1.md
Architecture-Product-Definition/account/authentication.policy.v1.md
Architecture-Product-Definition/out-links/out-link.model.specification.v1.md
Architecture-Product-Definition/analytics/analytics.model.specification.v1.md
```

---

## Forbidden Entities

The following entities are forbidden in Minime V1.

They must not be introduced in any table, Prisma model, repository, service, API, or event.

```
User              — forbidden. Account is the single identity root.
Profile           — forbidden. There is no separate Profile entity.
PublicProfile     — forbidden. There is no PublicProfile entity.
SocialAccount     — forbidden. Social Accounts domain owns no persistent storage.
```

There is no `profile_id` foreign key anywhere in the schema.

All account-owned data references `account_id`.

---

## Identifier Strategy

Every persistent entity uses UUID Version 7 as its primary key.

UUID v7 is time-ordered, globally unique, and system-generated.

Implementation rules:

- Generate identifiers in the backend at creation time.
- Identifiers never change after creation.
- Identifiers never encode business meaning.
- Foreign keys always reference primary identifiers.
- Sequential database integers must not be exposed publicly.

---

## Timestamp Rules

- All timestamps use UTC.
- Every primary entity contains `created_at` and `updated_at`.
- Soft-deletable entities additionally contain `deleted_at` (nullable). Exceptions: Account uses `status` enum for lifecycle; OutLink uses `status` + `archived_at`.
- `updated_at` is updated on every successful write.

---

## Soft Delete Policy

Default: soft delete using `deleted_at`.

Special cases (no `deleted_at` column):

- `Account` — uses `status` enum (`active | suspended | deleted`). Deletion is recorded via `status = 'deleted'`.
- `OutLink` — uses `status` enum (`created | active | archived`) with `archived_at` timestamp. Archived records are hard-deleted after 90 days by a retention job.
- `ConnectedAccount` — architecture specification requires Hard Delete.
- `UsernameReservation` — temporary record, physically deleted on expiry.
- `AnalyticsEvent` — append-only, deleted only by retention jobs acting on `created_at`.

Soft-deleted records are excluded from all queries by default.

---

## Entity Catalog

### 1. Account

**Owner:** Account domain

**Purpose:** The single canonical identity root. Represents one claimed Minime identity. There is no separate User entity. There is no separate Profile entity. Account is both.

**Prisma model name:** `Account`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | UUID | PK, NOT NULL | UUID v7, system-generated, immutable |
| `username` | TEXT | NOT NULL, UNIQUE | a-z/0-9, length 3–30, lowercase only, set at registration, not editable in V1 |
| `status` | AccountStatus | NOT NULL, DEFAULT 'active' | active \| suspended \| deleted |
| `settings` | JSONB | NOT NULL, DEFAULT '{}' | account preferences (notification prefs, etc.) |
| `qr_config` | JSONB | NOT NULL, DEFAULT '{}' | QR code configuration: `{color, format, storage_key}`. `storage_key` is the Object Storage key for the generated QR SVG; null until first generated. |
| `created_at` | TIMESTAMP | NOT NULL | UTC |
| `updated_at` | TIMESTAMP | NOT NULL | UTC, updated on every write |

**Constraints:**

- `UNIQUE (username)` — globally unique across all accounts
- `CHECK (status IN ('active', 'suspended', 'deleted'))`

**Indexes:**

- `username` — username lookup for public profile routing

**Lifecycle:**

- Created when registration completes (provider authentication completed)
- `status = 'suspended'` — administrative action, data preserved, sessions invalidated
- `status = 'deleted'` — user-initiated deletion, username permanently reserved

**What Account does NOT have:**

- No `user_id` column (no User entity)
- No `profile_id` column (no Profile entity)
- No `email` or `phone` column (authentication identities live in `AuthenticationIdentity`)
- No `display_name`, `bio`, or `avatar` columns (profile content lives in `ProfileContent`)

**settings field structure:**

```
settings = {
  gtm_container_id: string | null
}
```

- `gtm_container_id`: Google Tag Manager container ID provided by the account owner (e.g. `"GTM-XXXXXX"`). Null or absent means no GTM snippet is injected for this account's public profile. No other settings fields are supported in V1.

**qr_config field structure:**

```
qr_config = {
  color:       string | null,
  format:      string | null,
  storage_key: string | null
}
```

- `color`: QR foreground color (hex) or null for default.
- `format`: Always `'svg'` in V1.
- `storage_key`: Opaque Object Storage key for the generated QR SVG asset. Null until first QR is generated. Must never be exposed in API responses.

**QR generation rules:**
- QR is generated lazily on first `GET /api/v1/account/qr-code` if `storage_key` is null.
- `storage_key` is cleared when QR config changes via `updateQrCodeConfiguration`, forcing regeneration on next request.
- QR binary is owned by StoragePlatform. Only the storage key is persisted on Account.
- Account deletion must delete the QR storage asset if `storage_key` is set (handled by the `account.deletion.cascade` job via `AccountService.deleteQrAsset`).

---

### 2. AuthenticationIdentity

**Owner:** Authentication domain

**Purpose:** Represents a verified external identity provider link to an Account. V1 supports Google and Apple. Minime never authenticates users directly.

**Prisma model name:** `AuthenticationIdentity`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | UUID | PK, NOT NULL | UUID v7 |
| `account_id` | UUID | NOT NULL, FK → Account.id | owning account |
| `provider` | AuthProvider | NOT NULL | google \| apple |
| `provider_subject` | TEXT | NOT NULL | provider's canonical user identifier (Google `sub` or Apple `sub`) |
| `provider_email` | TEXT | NULLABLE | email reported by provider; informational only; not the uniqueness key |
| `created_at` | TIMESTAMP | NOT NULL | UTC |
| `updated_at` | TIMESTAMP | NOT NULL | UTC |

**Constraints:**

- `UNIQUE (provider, provider_subject)` — one Minime account per provider identity
- FK `account_id → Account.id`

**Indexes:**

- `(provider, provider_subject)` — auth lookup during sign-in
- `account_id` — list identities per account

**Notes:**

- Uniqueness is keyed by `(provider, provider_subject)`, not by email.
- `provider_email` is informational only. It may change over the provider identity's lifetime. It is never used as a uniqueness key or authentication identifier.
- No password hash. No OTP. Authentication is completed externally by the provider.
- Provider access tokens or refresh tokens are never stored here.

---

### 3. Session

**Owner:** Authentication domain

**Purpose:** An active authenticated session. Tracks the refresh token lifecycle.

**Prisma model name:** `Session`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | UUID | PK, NOT NULL | UUID v7 |
| `account_id` | UUID | NOT NULL, FK → Account.id | owning account |
| `refresh_token_hash` | TEXT | NOT NULL | hash of the issued refresh token |
| `expires_at` | TIMESTAMP | NOT NULL | refresh token expiry |
| `revoked_at` | TIMESTAMP | NULLABLE | set when explicitly revoked |
| `created_at` | TIMESTAMP | NOT NULL | UTC |
| `updated_at` | TIMESTAMP | NOT NULL | UTC, updated on token rotation |

**Constraints:**

- FK `account_id → Account.id`

**Indexes:**

- `account_id` — list active sessions per account
- `(account_id, revoked_at, expires_at)` — active session lookup

**Notes:**

- Access tokens are short-lived JWTs; they are not stored.
- Refresh tokens are rotated on every use: old hash invalidated, new hash stored.
- A revoked session has `revoked_at IS NOT NULL`.
- An expired session has `expires_at < NOW()`.

---

### 4. UsernameReservation

**Owner:** Username domain

**Purpose:** Temporary hold on a username during registration. Prevents two users from claiming the same username concurrently before the account is created.

**Prisma model name:** `UsernameReservation`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | UUID | PK, NOT NULL | UUID v7 |
| `username` | TEXT | NOT NULL, UNIQUE | reserved username, normalized to lowercase |
| `expires_at` | TIMESTAMP | NOT NULL | reservation TTL |
| `created_at` | TIMESTAMP | NOT NULL | UTC |

**Constraints:**

- `UNIQUE (username)` — enforces one active reservation per username

**Indexes:**

- `username` — availability check
- `expires_at` — cleanup job

**Deletion policy:** Hard Delete after expiry (cleanup job) or immediately after successful registration converts reservation into Account.

**Notes:**

- `UsernameReservation` is not Account-owned. The account does not exist yet.
- After successful registration, the reservation is deleted and the username is stored on `Account.username`.
- Username on Account is immutable; no updates to `Account.username` are supported in V1.

---

### 5. ProfileContent

**Owner:** Profile domain

**Purpose:** All user-authored profile content: display name, bio, avatar, contact information, and appearance configuration. There is no Profile entity. `account_id` is both the primary key and the foreign key.

**Prisma model name:** `ProfileContent`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `account_id` | UUID | PK, NOT NULL, FK → Account.id | one-to-one with Account |
| `display_name` | TEXT | NOT NULL | public name shown on profile |
| `bio` | TEXT | NULLABLE | max 1000 characters |
| `avatar_storage_key` | TEXT | NULLABLE | Object Storage key for avatar image |
| `contact` | JSONB | NOT NULL, DEFAULT '{}' | `{email, phone, whatsapp, location}` |
| `appearance_config` | JSONB | NOT NULL, DEFAULT '{}' | `{selected_theme_id, customizations}` |
| `created_at` | TIMESTAMP | NOT NULL | UTC |
| `updated_at` | TIMESTAMP | NOT NULL | UTC |

**Constraints:**

- `account_id` is PK — enforces one ProfileContent per Account
- FK `account_id → Account.id`

**No additional indexes needed** — `account_id` is the PK and is the only lookup key.

**Notes:**

- `ProfileContent` is created immediately when Account is created.
- There is no `profile_id`. `account_id` uniquely identifies the profile.
- There is no `status`, `published_at`, or `is_published` field. The profile has no publishing workflow.
- `appearance_config.selected_theme_id` references an immutable theme catalog defined in `packages/config/themes.ts` (not a persisted entity in this schema). Theme IDs are validated against this catalog at write time.
- Avatar binary content lives in Object Storage. Only the storage key is persisted here.

**Contact field structure:**

```
contact = {
  email:     string | null,
  phone:     string | null,
  whatsapp:  string | null,
  location:  string | null
}
```

**Appearance config field structure:**

```
appearance_config = {
  selected_theme_id:  string | null,
  customizations:     object
}
```

- `selected_theme_id`: ID of the active theme from the immutable theme catalog at `packages/config/themes.ts`. Null means the default theme is used.
- `customizations`: Reserved JSONB object for future theme-level overrides. **V1 scope:** `customizations` is always an empty object `{}` in V1. No customization fields are defined or accepted in V1. Unknown fields in this object must be ignored at read time and rejected at write time.

---

### 6. Block

**Owner:** Profile domain

**Purpose:** A single content unit on the profile. Blocks reference `account_id`, not `profile_id`.

**Prisma model name:** `Block`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | UUID | PK, NOT NULL | UUID v7 |
| `account_id` | UUID | NOT NULL, FK → Account.id | owning account |
| `type` | BlockType | NOT NULL | block type enum |
| `sort_order` | INTEGER | NOT NULL | ascending = display order |
| `content` | JSONB | NOT NULL, DEFAULT '{}' | type-specific user-authored content |
| `settings` | JSONB | NOT NULL, DEFAULT '{}' | type-specific user preferences |
| `style_overrides` | JSONB | NULLABLE | optional explicit style overrides for this block; null or absent means full inheritance from the active Theme; inherited values and resolved styles must not be stored here |
| `created_at` | TIMESTAMP | NOT NULL | UTC |
| `updated_at` | TIMESTAMP | NOT NULL | UTC |
| `deleted_at` | TIMESTAMP | NULLABLE | soft delete |

**Constraints:**

- FK `account_id → Account.id`

**Indexes:**

- `(account_id, deleted_at)` — list account's active blocks
- `(account_id, sort_order)` — ordered block retrieval

**Block types (V1):** `avatar | name | bio | image | social_icons | button | divider | title | textbox`

**Content field structure per block type:**

| BlockType | `content` fields | Source |
|---|---|---|
| `avatar` | `{}` | Reference block — data lives in `ProfileContent.avatar_storage_key` |
| `name` | `{}` | Reference block — data lives in `ProfileContent.display_name` |
| `bio` | `{}` | Reference block — data lives in `ProfileContent.bio` |
| `image` | `{ image_id: string }` | UUID of the owning `ImageAsset` record |
| `social_icons` | `{ accounts: [{ connected_account_id: string, sort_order: number }] }` | Reference IDs to ConnectedAccount records |
| `button` | `{ label: string, url: string }` | Owned directly by the block |
| `divider` | `{}` | No content — visual separator only |
| `title` | `{ text: string }` | Owned directly by the block |
| `textbox` | `{ text: string }` | Owned directly by the block |

**Settings field structure per block type:**

| BlockType | `settings` fields | Notes |
|---|---|---|
| `avatar` | `{ source: "profile_avatar" }` | Explicit reference declaration; immutable in V1 |
| `name` | `{ source: "profile_display_name" }` | Explicit reference declaration; immutable in V1 |
| `bio` | `{ source: "profile_bio" }` | Explicit reference declaration; immutable in V1 |
| `image` | `{}` | No block-specific settings in V1 |
| `social_icons` | `{}` | No block-specific settings in V1 |
| `button` | `{}` | No block-specific settings in V1 |
| `divider` | `{}` | No block-specific settings in V1 |
| `title` | `{}` | No block-specific settings in V1 |
| `textbox` | `{}` | No block-specific settings in V1 |

**Style overrides field structure:**

`style_overrides` is an optional JSONB field storing only the explicitly user-authored style keys for this block. It is distinct from `content` (user-authored data) and `settings` (type-specific user preferences).

```
style_overrides?: Record<string, unknown>
```

Rules:
- Null or absent: the block inherits all style values from the active Theme. No override entry is written.
- Present: only explicitly overridden keys are stored. Inherited values must not be stored.
- Resolved styles (active Theme merged with block `style_overrides`) are computed at render time by the Renderer. Resolved styles must not be stored as canonical block data.
- The Renderer must not read raw theme defaults or raw block overrides directly. It must consume only the resolved style object produced by combining the active Theme with the block's `style_overrides`.

---

**Instance rules:**

| Category | Block Types | Limit |
|---|---|---|
| Identity blocks | `avatar`, `name`, `bio` | Max 1 per account each |
| All blocks (total) | All types combined | MAX_BLOCKS_PER_ACCOUNT per account |

**MAX_BLOCKS_PER_ACCOUNT** (canonical definition — implementation default: 100). This is a V1 operational safety limit, not a pricing feature. Block creation must be rejected when total active block count reaches this limit. Changing the default value does not require architectural redesign. Do not introduce plan tiers.

---

### 7. ImageAsset

**Owner:** Profile domain

**Purpose:** Stores metadata for an image uploaded for use in an Image Block. Each `ImageAsset` record connects a binary in Object Storage to the `image_id` field in an image block's `content`. This is not a media library — there is no search, browse, gallery, album, or folder capability.

**Prisma model name:** `ImageAsset`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | UUID | PK, NOT NULL | UUID v7, system-generated, immutable |
| `account_id` | UUID | NOT NULL, FK → Account.id | owning account |
| `storage_key` | TEXT | NOT NULL | opaque Object Storage key returned by StoragePlatform |
| `mime_type` | TEXT | NOT NULL | canonical MIME type after storage processing |
| `width` | INTEGER | NOT NULL | pixel width of the stored image |
| `height` | INTEGER | NOT NULL | pixel height of the stored image |
| `file_size` | INTEGER | NOT NULL | file size in bytes of the stored (processed) asset |
| `created_at` | TIMESTAMP | NOT NULL | UTC |

**No `updated_at`.** `ImageAsset` is immutable after creation.

**No `deleted_at`.** `ImageAsset` uses Hard Delete.

**Constraints:**

- FK `account_id → Account.id`

**Indexes:**

- `account_id` — list ImageAssets per account; used during account deletion cascade

**Notes:**

- `ImageAsset` exists solely to support Image Block rendering. It is not a general-purpose media store.
- The `image_id` field in an image block's `content` is the UUID of the referenced `ImageAsset`.
- Binary content lives in Object Storage. Only the storage key and derived metadata are persisted here.
- Storage keys are never exposed in API responses; public URLs are constructed at render time via `StoragePlatform.buildPublicUrl`.
- Orphaned `ImageAsset` records (blocks soft-deleted without image cleanup) are purged by the account deletion cascade.

---

### 8. ConnectedAccount

**Owner:** Connected Accounts domain

**Purpose:** Persistent storage for user-entered social links. This is NOT an OAuth record. This is NOT a provider connection. This is a lightweight record of `{platform, username, url}` that the user has added to their profile.

**Prisma model name:** `ConnectedAccount`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | UUID | PK, NOT NULL | UUID v7 |
| `account_id` | UUID | NOT NULL, FK → Account.id | owning account |
| `platform` | SocialPlatform | NOT NULL | platform enum |
| `username` | TEXT | NOT NULL | platform username, normalized |
| `url` | TEXT | NOT NULL | canonical URL, system-generated from platform rules |
| `sort_order` | INTEGER | NOT NULL | user-controlled display order |
| `created_at` | TIMESTAMP | NOT NULL | UTC |
| `updated_at` | TIMESTAMP | NOT NULL | UTC |

**No `deleted_at` column — ConnectedAccount uses Hard Delete.**

**Constraints:**

- FK `account_id → Account.id`
- `UNIQUE (account_id, platform, username)` — prevents duplicate entries for the same identity

**Indexes:**

- `(account_id, sort_order)` — ordered list per account
- `(account_id, platform)` — filter by platform

**Notes:**

- The Social Accounts domain (processing) produces normalized records and hands them off.
- The Connected Accounts domain stores them. Social Accounts owns no persistent storage.
- `url` is always system-generated using platform rules. Users never provide URLs directly.
- Editing: only `username` is editable. Platform changes require delete + create. `sort_order` changes through reorder operations, not field edits.
- Deletion is permanent (Hard Delete). There is no trash, archive, or undo.
- There is no `status`, `is_verified`, `connection_status`, `last_synced_at`, or `revoked_at`.
- No OAuth tokens, provider metadata, or access credentials are stored here.

---

### 9. OutLink

**Owner:** Out Links domain

**Purpose:** A managed outbound destination representing one clickable state. References `account_id`, not `profile_id`. Stores an immutable snapshot of the source block's state at creation time.

**Prisma model name:** `OutLink`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | UUID | PK, NOT NULL | UUID v7, internal identifier, never exposed publicly |
| `public_id` | TEXT | NOT NULL, UNIQUE | URL-safe public identifier used in `/out/{public_id}` routing |
| `account_id` | UUID | NOT NULL, FK → Account.id | owning account |
| `block_id` | UUID | NOT NULL | source block reference, immutable (no FK enforcement) |
| `block_type` | TEXT | NOT NULL | source block type at creation time, immutable |
| `destination_url` | TEXT | NOT NULL | resolved external URL, immutable snapshot |
| `label_snapshot` | TEXT | NULLABLE | label text captured at creation time, immutable |
| `source_snapshot` | JSONB | NOT NULL | full snapshot of source block state at creation time, immutable |
| `connected_account_id` | TEXT | NULLABLE | ConnectedAccount ID for social_icons block OutLinks; null for button block OutLinks; immutable |
| `status` | OutLinkStatus | NOT NULL, DEFAULT 'active' | created \| active \| archived |
| `archived_at` | TIMESTAMP | NULLABLE | set when status = 'archived'; null while active |
| `created_at` | TIMESTAMP | NOT NULL | UTC |
| `updated_at` | TIMESTAMP | NOT NULL | UTC |

**No `deleted_at` column. No `is_enabled` field. Lifecycle is managed through `status`.**

**Constraints:**

- FK `account_id → Account.id`
- `UNIQUE (public_id)` — globally unique public identifier

**Indexes:**

- `(account_id, status)` — list active out links per account
- `public_id` — `/out/{public_id}` resolution
- `(account_id, created_at)` — ordered list
- `(block_id, connected_account_id)` — rendering lookup: resolves active OutLink per clickable icon within a social_icons block, or per button block

**Lifecycle:**

- `created → active → archived → purge (hard delete after 90 days from archived_at)`

**Notes:**

- OutLink is separate from the `button` block. A button block embeds a URL directly. An OutLink is a managed redirect with click tracking.
- `destination_url`, `label_snapshot`, `source_snapshot`, `block_id`, `block_type`, `public_id`, `connected_account_id` are all immutable after creation.
- A `button` block creates exactly one OutLink (`connected_account_id` = null). A `social_icons` block creates one OutLink per entry in `content.accounts`, with `connected_account_id` set to the corresponding ConnectedAccount ID.
- When a ConnectedAccount is removed from a social_icons block, its OutLink is archived. When a social_icons block is deleted, all its OutLinks are archived.
- Raw social destination URLs must not appear in public render payloads. Rendering always uses `/out/{public_id}` per icon.
- If no active OutLink exists for a button block or for a specific social icon, that link is omitted from the rendered output.
- Public outbound navigation always executes through the Out Links domain (never directly).
- The internal `id` is never exposed publicly. Only `public_id` appears in public routes.

---

### 10. AnalyticsEvent

**Owner:** Analytics domain

**Purpose:** Immutable record of an observed event. Append-only. V1 stores only two event types: `profile_view` and `link_click`. References to business entities are reference-only — not enforced foreign keys.

**Prisma model name:** `AnalyticsEvent`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | UUID | PK, NOT NULL | UUID v7 |
| `event_type` | AnalyticsEventType | NOT NULL | profile_view \| link_click |
| `account_id` | UUID | NULLABLE | profile_view only — owning account (no FK enforcement) |
| `out_link_id` | UUID | NULLABLE | link_click only — clicked out link (no FK enforcement) |
| `device_category` | DeviceCategory | NULLABLE | profile_view only — desktop \| mobile \| tablet |
| `created_at` | TIMESTAMP | NOT NULL | UTC, event timestamp |

**No `updated_at`. No `deleted_at`. AnalyticsEvent is append-only.**

**Constraints:**

- No foreign key enforcement on `account_id` or `out_link_id`. Entity deletion must not cascade to analytics.

**Indexes:**

- `(event_type, created_at)` — time-range queries by event type
- `(account_id, created_at)` — profile analytics per account
- `(out_link_id, created_at)` — link click analytics
- `device_category` — device breakdown

**Event shapes:**

- `profile_view` → `account_id` required, `device_category` required, `out_link_id` null
- `link_click` → `out_link_id` required, `account_id` null, `device_category` null

**V1 non-goals (not stored):** visitor identity, geographic data, referrer data, browser data, OS data.

**Deletion policy:** Records are deleted only by retention jobs. No record is manually deleted.

---

## Enum Definitions

### AccountStatus

```
active      — account operates normally
suspended   — restricted by administrative action, data preserved
deleted     — deleted by user, data preserved for audit integrity
```

### AuthProvider

```
google      — Google Sign-In (ID token verification against Google public keys)
apple       — Apple Sign-In (ID token verification against Apple public keys)
```

### BlockType

```
avatar
name
bio
image
social_icons
button
divider
title
textbox
```

### SocialPlatform

```
instagram
tiktok
youtube
linkedin
x
threads
facebook
snapchat
spotify
telegram
twitch
behance
```

Platform-specific validation rules (handle format, URL template) live in:
```
Architecture-Product-Definition/social-accounts/social-platforms/
```

### AnalyticsEventType

```
profile_view    — a visitor opened the public profile
link_click      — a visitor clicked a managed out link
```

### DeviceCategory

```
desktop
mobile
tablet
```

### OutLinkStatus

```
created     — initial state on record creation
active      — out link is publicly accessible
archived    — out link is archived; hard-deleted after 90 days
```

---

## Entity Relationship Map

```
Account
  │
  ├─── AuthenticationIdentity (1:many, account_id FK)
  │
  ├─── Session             (1:many, account_id FK)
  │
  ├─── ProfileContent      (1:1, account_id is PK)
  │
  ├─── Block               (1:many, account_id FK)
  │       └── content.image_id → ImageAsset.id  (logical reference, no DB FK)
  │
  ├─── ImageAsset          (1:many, account_id FK, Hard Delete)
  │
  ├─── ConnectedAccount    (1:many, account_id FK, Hard Delete)
  │
  ├─── OutLink             (1:many, account_id FK)
  │
  └─── AnalyticsEvent      (reference only, no FK enforcement)

UsernameReservation         (standalone, no Account FK — account does not exist yet)
```

---

## What Is Not Persisted

The following exist in the architecture as catalog assets, processing logic, or derived data — not as database entities.

| Concept | Storage Location | Notes |
|---|---|---|
| Theme definitions | Application code / config files | Immutable catalog, not user data |
| Social platform rules | Application code | URL templates, validation rules |
| SEO metadata | Computed at render time | Derived from `ProfileContent` + `Account` |
| Integration configurations | Application config | Provider registration, loading rules |
| Rendered profile | Cache (Redis) | Derived from stored entities, TTL ≤ 60s |
| Resolved block styles | Computed at render time | Derived from active Theme merged with `Block.style_overrides`; resolved values must not be stored as canonical block data |
| Access tokens | Not stored | Short-lived JWTs, not persisted |

---

## Cascade Rules

Database-level cascade deletion is not used for business lifecycle behavior.

- Deleting Account does not cascade-delete child records.
- Child record deletion (blocks, connected accounts, out links) follows domain-specific business rules owned by each Product Domain service.

---

## Transaction Requirements

The following operations require a database transaction:

| Operation | Entities Involved |
|---|---|
| Complete registration | `UsernameReservation` delete + `Account` create + `AuthenticationIdentity` create + `ProfileContent` create |
| Block reorder | Multiple `Block` sort_order updates |
| Account deletion | `AccountService.deleteAccount` executes two steps: (1) `Account.status = 'deleted'`; (2) publishes `account.deleted` event. Sessions are invalidated synchronously via in-process EventEmitter handler (AuthService.revokeAllSessions). A second EventEmitter handler enqueues the `account.deletion.cascade` BullMQ job for durable cross-domain cleanup. The cascade job executes in order: (1) collect out_link_ids via OutLinksService.getAccountOutLinkIds (read-only, before any deletion); (2) AnalyticsEvents hard-deleted (profile_view by account_id; link_click by out_link_ids) — analytics deleted before OutLinks to guarantee idempotency on retry; (3) OutLinks hard-deleted; (4) ProfileContent hard-deleted (includes `appearance_config` JSONB field); (5) Blocks hard-deleted; (6) ImageAssets hard-deleted (StoragePlatform.delete per asset); (7) ConnectedAccounts hard-deleted; (8) Binary avatar asset physically deleted; (9) QR storage asset deleted if `Account.qr_config.storage_key` is set (AccountService.deleteQrAsset). The Account record is retained permanently. No separate QR entity (`qr_config` is a JSONB field on Account, retained with Account). No AppearanceState entity (`appearance_config` is a JSONB field on ProfileContent, deleted with ProfileContent). No AI decisions entity in V1. Cascade order and idempotency rules: see `10-background-jobs.md` — Job 7. |

---

## Repository Access Rules

- Repositories exclude soft-deleted records by default.
- Repository ownership follows Product Domain ownership:

| Repository | Owner Domain |
|---|---|
| AccountRepository | Account |
| AuthRepository (Session, AuthenticationIdentity) | Authentication |
| UsernameReservationRepository | Username |
| ProfileContentRepository | Profile |
| BlockRepository | Profile |
| ImageAssetRepository | Profile |
| ConnectedAccountRepository | Connected Accounts |
| OutLinkRepository | Out Links |
| AnalyticsEventRepository | Analytics |
