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
- Soft-deletable entities additionally contain `deleted_at` (nullable). Exceptions: Account uses `status` enum for lifecycle; OutLink uses `status` + `archived_at`; AnalysisSession uses `status` enum and is Hard Delete only (no `deleted_at`).
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
- `AnalysisSession` — Hard Delete only, executed by the account deletion cascade. No retention job exists in V1.
- `AccountQRCode` — Hard Delete only, executed by the account deletion cascade. No retention job exists in V1; the record is never user-deletable.

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
| `created_at` | TIMESTAMP | NOT NULL | UTC |
| `updated_at` | TIMESTAMP | NOT NULL | UTC, updated on every write |

**Constraints:**

- `UNIQUE (username)` — globally unique across all accounts
- `CHECK (status IN ('active', 'suspended', 'deleted'))`

**Indexes:**

- `username` — username lookup for public profile routing

**Lifecycle:**

- Created when registration completes (provider authentication completed)
- `status = 'active'` — must never be reached unless the Account's `AccountQRCode` record (entity 12) and its canonical SVG asset have already been successfully created and persisted; see entity 12 — Notes
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

**QR Code is not a field on Account.** It is a separate Account-owned entity, `AccountQRCode` (entity 12 below), uniquely keyed by `account_id`. Account never embeds QR configuration, customization, or storage keys directly.

---

### 2. AuthenticationIdentity

**Owner:** Authentication domain

**Purpose:** Represents a verified external identity provider link to an Account. V1 supports Google and Apple. Minime never authenticates users directly.

**Prisma model name:** `AuthenticationIdentity`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `auth_identity_id` | UUID | PK, NOT NULL | UUID v7, system-generated, permanent, immutable |
| `account_id` | UUID | NOT NULL, FK → Account.id | owning account |
| `provider` | AuthProvider | NOT NULL | `google` \| `apple` only in V1 — no other value is valid; extensible by adding adapters and allowlist entries in a future version, never by changing this column's type shape |
| `provider_subject` | TEXT | NOT NULL | provider's canonical user identifier (Google `sub` or Apple `sub`); immutable after creation; the only authentication identity key |
| `email` | TEXT | NULLABLE | email reported by provider, if any; optional; informational only; never unique; never the login identity; never used for automatic account merge |
| `email_verified` | BOOLEAN | NOT NULL, DEFAULT false | whether the provider reported the email as verified; informational only |
| `provider_profile` | JSONB | NULLABLE | generic, optional, non-authoritative provider-returned metadata (e.g. `display_name`, `avatar_url`, `locale`, or normalized raw claims); never used for ownership, authorization, uniqueness, or identity matching; never replaces Profile Content |
| `created_at` | TIMESTAMP | NOT NULL | UTC |
| `updated_at` | TIMESTAMP | NOT NULL | UTC; updated only when provider-reported metadata (`email`, `email_verified`, `provider_profile`) is refreshed on a later authentication — `provider_subject` itself never changes |

**There is no `display_name` column on this entity.** Any provider-reported display name is one possible key inside `provider_profile`, never a dedicated top-level field.

**Constraints:**

- `UNIQUE (provider, provider_subject)` — globally unique; one Minime account per provider identity; no two accounts may share a `(provider, provider_subject)` pair
- FK `account_id → Account.id`
- `CHECK (provider IN ('google', 'apple'))` — the V1 allowlist; future providers are added by extending this constraint and adding an adapter, never by restructuring the entity

**Indexes:**

- `(provider, provider_subject)` — auth lookup during sign-in and duplicate-identity detection
- `account_id` — list identities per account; used to enforce "at least one identity per account"

**Notes:**

- Uniqueness is keyed by `(provider, provider_subject)`, never by `email`.
- `email` is optional, is never unique, and is never used as a uniqueness key, authentication identifier, or automatic-merge signal. It is provider-returned profile metadata only. See "Identity Merge Policy" in `authentication.policy.v1.md`.
- `provider_subject` is immutable once set. There is no "change identity" operation — an identity is linked (created) or unlinked (deleted), never edited in place beyond its metadata fields (`email`, `email_verified`, `provider_profile`).
- `provider_profile` stores non-authoritative, provider-returned metadata only (e.g. `{ "display_name": "...", "avatar_url": "...", "locale": "..." }` or normalized raw claims). It must never be used for account ownership, authorization, uniqueness, or identity matching, and it must never replace `ProfileContent`, which remains user-owned and authoritative for public profile display.
- Apple's one-time `name`/`email` payload and Apple private relay emails are valid optional metadata only, captured into `email`/`provider_profile` at the moment they are returned.
- Google's `email`, display name, and avatar are valid optional metadata only.
- No password hash. No OTP fields of any kind. Authentication is completed externally by the provider.
- Provider access tokens or refresh tokens are never stored here, and are not persisted anywhere in V1 unless a future provider explicitly requires it and this data model and the security model are both updated accordingly.
- Every Account must have at least one `AuthenticationIdentity` row at all times. The last remaining row for an Account must never be deleted (enforced by `AuthService.unlinkAuthenticationProvider`; see `04-service-contracts.md`).

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

- `ProfileContent` is created immediately when Account is created, in the same registration transaction (see `04-service-contracts.md` — AccountService.createAccount / AuthService "Registration Transaction Coordination"). `display_name` is initialized to the reserved username at creation time — this is the only source rule for its initial value; provider-reported display names (`AuthenticationIdentity.provider_profile`) are never used to populate it. `display_name` is never empty, either at creation or thereafter, and is user-editable from that point on.
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
- `UNIQUE (account_id, type) WHERE type IN ('avatar', 'name', 'bio') AND deleted_at IS NULL` — enforces the "max 1 identity block per account" invariant at the database level; a concurrent double-`addBlock` race for the same identity type fails this constraint rather than silently producing two active identity blocks of the same type.
- `UNIQUE (account_id, sort_order) WHERE deleted_at IS NULL` — no two active blocks for the same account may share a `sort_order` value. Combined with the application-level lock below, this makes duplicate `sort_order` unreachable even under concurrent `addBlock`/`reorderBlocks` calls for the same account.

**Concurrency:** `addBlock`, `updateBlock`, `deleteBlock`, and `reorderBlocks` each acquire a Postgres advisory transaction lock keyed by `account_id` (`pg_advisory_xact_lock(hashtext(account_id))`) before reading or writing Block rows for that account, and hold it for the duration of their transaction. This serializes block-mutating operations per account (the only realistic concurrent-access case is the same user in two browser tabs), which is what makes `MAX_BLOCKS_PER_ACCOUNT` enforcement (read count, then insert) and `sort_order` assignment race-free without weakening the platform's Last Write Wins concurrency model for ordinary field values (`data.architecture.specification.v1.md` — "Concurrency") — the lock protects invariants (count, uniqueness), not field-value conflict resolution, which remains LWW.

**Indexes:**

- `(account_id, deleted_at)` — list account's active blocks
- `(account_id, sort_order)` — ordered block retrieval; the partial unique constraint above rides on this same column pair

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
- `UNIQUE (account_id, sort_order)` — no two ConnectedAccount rows for the same account may share a `sort_order` value (ConnectedAccount uses Hard Delete, so there is no `deleted_at` to condition on, unlike Block)

**Concurrency:** `addConnectedAccount`, `updateConnectedAccount`, `removeConnectedAccount`, and `reorderConnectedAccounts` each acquire the same per-account Postgres advisory transaction lock described under Block ("Concurrency" above), keyed identically by `account_id`. This is the same lock, not a second independent one — Block and ConnectedAccount mutations for the same account are serialized against each other too, which matters because `removeConnectedAccount` also writes to `BlockRepository` (social_icons `content.accounts`) within its transaction (`04-service-contracts.md`).

**Indexes:**

- `(account_id, sort_order)` — ordered list per account; the unique constraint above rides on this same column pair
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
- `UNIQUE (block_id, connected_account_id) WHERE status = 'active'` — enforces "at most one active OutLink per clickable identity" at the database level, independent of and in addition to application-level transaction ordering in `04-service-contracts.md` (ProfileService.updateBlock / ConnectedAccountsService). A concurrent double-create attempt fails this constraint rather than silently producing two active OutLinks for the same button or social icon.

**Indexes:**

- `(account_id, status)` — list active out links per account
- `public_id` — `/out/{public_id}` resolution
- `(account_id, created_at)` — ordered list
- `(block_id, connected_account_id)` — rendering lookup: resolves active OutLink per clickable icon within a social_icons block, or per button block; the partial unique constraint above rides on this same column pair

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
| `id` | UUID | PK, NOT NULL | `profile_view`: UUID v7 generated at insert. `link_click`: the caller-supplied `click_id` generated by `OutLinksService.resolveOutLink` at the moment of redirect — this is the click idempotency key; the PK constraint itself rejects duplicate-click retries (see 04-service-contracts.md — AnalyticsService.recordLinkClick) |
| `event_type` | AnalyticsEventType | NOT NULL | profile_view \| link_click |
| `account_id` | UUID | NULLABLE | profile_view: owning account. link_click: an immutable snapshot of `OutLink.account_id` captured at click time — not a live reference, and not re-derived after the source OutLink is purged. No FK enforcement in either case. |
| `out_link_id` | UUID | NULLABLE | link_click only — clicked out link (no FK enforcement) |
| `device_category` | DeviceCategory | NULLABLE | profile_view only — desktop \| mobile \| tablet |
| `created_at` | TIMESTAMP | NOT NULL | UTC, event timestamp |

**No `updated_at`. No `deleted_at`. AnalyticsEvent is append-only.**

**Constraints:**

- No foreign key enforcement on `account_id` or `out_link_id`. Entity deletion must not cascade to analytics.
- `PRIMARY KEY (id)` doubles as the click-deduplication constraint for `link_click`: since `id` is the caller-supplied `click_id` rather than a fresh UUID, re-attempting the same click (job retry) always collides on this key rather than inserting a duplicate row.

**Indexes:**

- `(event_type, created_at)` — time-range queries by event type
- `(account_id, created_at)` — profile analytics per account, and account-level click totals for link_click (now possible without joining OutLink, since account_id is snapshotted directly — see "Event shapes" below)
- `(out_link_id, created_at)` — link click analytics
- `device_category` — device breakdown

**Event shapes:**

- `profile_view` → `account_id` required, `device_category` required, `out_link_id` null
- `link_click` → `out_link_id` required, `account_id` required (snapshot, immutable, survives OutLink purge), `device_category` null

**V1 non-goals (not stored):** visitor identity, geographic data, referrer data, browser data, OS data.

**Deletion policy:** Records are deleted only by retention jobs. No record is manually deleted.

---

### 11. AnalysisSession

**Owner:** AI Platform

**Purpose:** A persisted record of one on-demand "Analyze My Profile" execution. The Analysis Session is an official V1 entity — not merely a response payload. It stores one completed AI analysis result and is the unit of Analysis Session Reuse.

**Prisma model name:** `AnalysisSession`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `id` | UUID | PK, NOT NULL | UUID v7 |
| `account_id` | UUID | NOT NULL, FK → Account.id | owning account |
| `input_hash` | TEXT | NOT NULL | hash of the normalized Analysis Input Snapshot (account, profile content, connected accounts, blocks, appearance, analytics summary) |
| `analysis_version` | TEXT | NOT NULL | semantic version string (`MAJOR.MINOR.PATCH`, e.g. `"1.0.0"`) of Minime's analysis business logic (prompt meaning, scoring, recommendation logic, output interpretation, input rules) used to produce this session; owned by Minime, not the Provider |
| `output_schema_version` | TEXT | NOT NULL | semantic version string (`MAJOR.MINOR.PATCH`, e.g. `"1.0.0"`) of the structured output schema used to validate and store the AI response; owned by Minime, not the Provider |
| `status` | AnalysisSessionStatus | NOT NULL, DEFAULT 'pending' | pending \| completed \| failed |
| `provider_key` | TEXT | NULLABLE | diagnostic metadata only — configured provider identifier at execution time, for debugging/support/monitoring/cost analysis; null until execution starts; must never affect reuse, validity, or any business decision |
| `model_key` | TEXT | NULLABLE | diagnostic metadata only — configured model identifier at execution time, for debugging/support/monitoring/cost analysis; null until execution starts; must never affect reuse, validity, or any business decision |
| `report` | JSONB | NULLABLE | structured analysis report; null until status = 'completed' |
| `scores` | JSONB | NULLABLE | structured scoring output; null until status = 'completed' |
| `suggestions` | JSONB | NULLABLE | profile improvement suggestions; null until status = 'completed' |
| `recommendations` | JSONB | NULLABLE | recommended actions; null until status = 'completed' |
| `metadata` | JSONB | NOT NULL, DEFAULT '{}' | implementation-defined execution metadata (e.g. timing, token counts) |
| `created_at` | TIMESTAMP | NOT NULL | UTC |
| `updated_at` | TIMESTAMP | NOT NULL | UTC, updated when status transitions |
| `completed_at` | TIMESTAMP | NULLABLE | UTC; set when status becomes 'completed' or 'failed' |

**No `deleted_at` column. AnalysisSession uses Hard Delete only, executed by the account deletion cascade.**

**Constraints:**

- FK `account_id → Account.id`

**Indexes:**

- `(account_id, status, created_at)` — find the latest completed Analysis Session for an account
- `(account_id, input_hash, analysis_version, output_schema_version, status)` — full Analysis Session Reuse lookup

**Notes:**

- An Analysis Session belongs to exactly one Account.
- An Analysis Session is created only after an explicit "Analyze My Profile" user request. There is no background or scheduled creation.
- Only `status = 'completed'` Analysis Sessions may be reused. Failed Analysis Sessions must never be reused.
- **Final Analysis Session Reuse lookup:** find the latest `AnalysisSession` where `account_id` = current account, `status` = `'completed'`, `input_hash` = current input hash, `analysis_version` = current configured analysis version (exact string match), and `output_schema_version` = current configured output schema version (exact string match). Only that session may be reused. A mismatch on any one of `input_hash`, `analysis_version`, or `output_schema_version` requires creating a new Analysis Session. Reuse never depends on `created_at` age.
- `analysis_version` is a semantic version string. It changes only when Minime's analysis business meaning changes (prompt meaning, scoring logic, recommendation logic, structured output interpretation, analysis input rules). It is not a model version and not a provider version; changing the configured Provider or Model never changes it automatically.
- `output_schema_version` is a semantic version string. It changes only when the structured output JSON shape changes. It is not a model version and not a provider version.
- `provider_key` and `model_key` are diagnostic metadata only (debugging, support, monitoring, cost analysis, provider performance review, incident investigation). They must never participate in the Analysis Session Reuse lookup, analysis validity, AI output validation, or any business decision. A change to the configured Provider or Model never invalidates an existing Analysis Session by itself — invalidation requires explicitly incrementing `analysis_version` or `output_schema_version`.
- Analysis Session history is not AI learning. It stores reports only. There is no `AiDecision`, `AcceptedDecision`, or `SuggestionHistory` entity in V1; accepted suggestions are persisted as ordinary Product Domain data by the responsible domain (e.g. `ProfileContent.bio`).

---

### 12. AccountQRCode

**Owner:** Account domain

**Purpose:** The Account's single, permanent QR Code record. QR Code is an implementation area within the Account domain, never a standalone Product Domain. References the canonical SVG asset and the Account's QR redirect route. Full architectural definition: `Architecture-Product-Definition/qr-code/qr-code.system.specification.v1.md`.

**Prisma model name:** `AccountQRCode`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `qr_code_id` | UUID | PK, NOT NULL | UUID v7, system-generated, globally unique, permanent, immutable; this is the QR Code's identity |
| `account_id` | UUID | NOT NULL, UNIQUE, FK → Account.id | owning account; never the username; immutable after creation |
| `qr_url` | TEXT | NOT NULL | always equals `https://minime.ae/qr/{qr_code_id}`; immutable after creation |
| `destination_type` | TEXT | NOT NULL, DEFAULT 'public_profile' | always `'public_profile'` in V1; no other destination type exists |
| `canonical_asset_id` | TEXT | NOT NULL | references the stored SVG asset (Storage Platform key/identifier); must exist before the owning Account becomes active |
| `canonical_format` | TEXT | NOT NULL, DEFAULT 'svg' | always `'svg'`; PNG is never the canonical stored format |
| `created_at` | TIMESTAMP | NOT NULL | UTC; set once at creation |
| `updated_at` | TIMESTAMP | NOT NULL | UTC; updated only by system-triggered asset regeneration (recovery/migration), never by the user |

There is no username-snapshot field of any name, and no other field that stores `username` or any value derived from it. `username` is never persisted on this entity.

**No `deleted_at` column. AccountQRCode uses Hard Delete only, executed by the account deletion cascade.**

**Constraints:**

- `PRIMARY KEY (qr_code_id)` — `qr_code_id` is globally unique; no two QR Code records may share the same `qr_code_id`
- `UNIQUE (account_id)` — enforces exactly one QR Code record per Account; no two QR Code records may reference the same Account
- FK `account_id → Account.id`

Combined, these two constraints enforce the bidirectional invariant: `One Account = One QR Code Record` and `One QR Code Record = One Account`.

**Indexes:**

- `account_id` — unique lookup (covered by the `UNIQUE (account_id)` constraint)
- `qr_code_id` — public route resolution lookup (`GET /qr/{qr_code_id}`); covered by the PK

**Notes:**

- `One Account = One QR Code Record` and `One QR Code Record = One Account` are permanent invariants, enforced respectively by `UNIQUE(account_id)` and `PRIMARY KEY(qr_code_id)`.
- The record is keyed by `account_id`. It is never keyed by `username`.
- `qr_code_id` is globally unique, permanent, and immutable. It is never regenerated, reassigned, or reused.
- **No username storage:** This entity never stores `username` or a username snapshot. The QR redirect flow always resolves the destination live: `qr_code_id → AccountQRCode → account_id → Account.username → redirect to /{username}`. This avoids duplicating Account data and remains correct regardless of any future username-change capability.
- **Mandatory before activation:** The record, together with its `canonical_asset_id` SVG asset, must be successfully created and persisted before the owning Account reaches `status = 'active'`. If either creation step fails, the Account must not become active. There is no lazy-creation path triggered by a later dashboard view, settings view, download, share, or public profile visit (see `04-service-contracts.md` — AccountService, and `13-implementation-plan.md` — Phase 4).
- The user must never edit, customize, regenerate, replace, reset, or delete this record. Only the system may regenerate the underlying asset (`canonical_asset_id`), and only for missing/corrupted asset recovery, storage migration, or format migration — and only by updating `canonical_asset_id`/`updated_at` while preserving `qr_code_id`, `account_id`, and `qr_url`.
- `canonical_asset_id` is generated, not user-uploaded; it is stored through the Storage Platform per `11-platform-services.md`.
- PNG export/download is computed on demand from the canonical SVG asset and must never be persisted as a second `AccountQRCode` record or as a replacement `canonical_format`.

---

### 13. AccountDeletionOutbox

**Owner:** Account domain

**Purpose:** Guarantees that cross-domain account-deletion cascade cleanup is eventually triggered even if the process crashes between committing `Account.status = 'deleted'` and the best-effort `account.deleted` EventEmitter notification firing. This is the durability mechanism required because, per `Architecture-Product-Definition/platform/events/events.architecture.specification.v1.md` ("Failure Philosophy") and `event.lifecycle.specification.v1.md` ("Product correctness always has higher priority than Event recording"), business correctness must never depend solely on EventEmitter delivery. This entity is not part of the Events Platform; it is a narrow, single-purpose reliability primitive scoped only to the `deleteAccount` command.

**Prisma model name:** `AccountDeletionOutbox`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `account_id` | UUID | PK, NOT NULL, FK → Account.id | one row per deleted account; created once, at deletion |
| `requested_at` | TIMESTAMP | NOT NULL | UTC; set when the row is inserted, in the same transaction as `Account.status = 'deleted'` |
| `dispatched_at` | TIMESTAMP | NULLABLE | UTC; set once the Deletion Outbox Dispatcher job has successfully enqueued `account.deletion.cascade` for this account |
| `attempts` | INTEGER | NOT NULL, DEFAULT 0 | incremented by the dispatcher on every processing attempt, for operational visibility only — it never gates retry |

**Constraints:**

- `PRIMARY KEY (account_id)` — at most one outbox row per account
- FK `account_id → Account.id`

**Indexes:**

- `dispatched_at` (partial, `WHERE dispatched_at IS NULL`) — used by the Deletion Outbox Dispatcher job to find undispatched rows

**Notes:**

- Inserted by `AccountService.deleteAccount` in the same database transaction that sets `Account.status = 'deleted'` — see `04-service-contracts.md`. Never inserted by any other command.
- Read and updated only by the Deletion Outbox Dispatcher background job — see `10-background-jobs.md` — Job 8.
- Rows are never deleted. Retained as a permanent record that the account's cascade cleanup was reliably triggered, mirroring the permanent retention of the `Account` row itself.
- A row with `dispatched_at IS NULL` and `requested_at` older than 1 hour indicates the dispatcher itself is failing systemically and must page an operator; this is the reconciliation mechanism for stuck deletions — no separate monitoring subsystem exists or is required.

---

### 14. AuditLog

**Owner:** Platform (cross-domain; written by Account, Authentication, Connected Accounts, and Profile domains). Full policy: `Architecture-Product-Definition/account-management/audit.logging.policy.v1.md`.

**Purpose:** Internal, non-product-facing operational record of meaningful account, authentication, connected-account, profile, and administrative changes, for security investigation, support, and operational purposes only. Not a versioning or rollback system. Not visible to end users.

**Prisma model name:** `AuditLog`

**Fields:**

| Field | Type | Constraints | Notes |
|---|---|---|---|
| `log_id` | UUID | PK, NOT NULL | UUID v7, system-generated, globally unique, immutable, permanent |
| `account_id` | UUID | NOT NULL, FK → Account.id | the account the event concerns; no `ON DELETE CASCADE` — the row survives account deletion for the retention period, exactly like the `Account` row itself |
| `actor_account_id` | UUID | NULLABLE, FK → Account.id | populated only for administrative actions performed by staff acting on another account; NULL when the account acted on itself |
| `action` | TEXT | NOT NULL | one value from the closed catalog in "Logged Event Catalog" below |
| `entity_type` | TEXT | NOT NULL | one of: `account` \| `authentication_identity` \| `connected_account` \| `profile` \| `block` \| `admin` |
| `entity_id` | TEXT | NOT NULL | identifier of the affected entity |
| `old_value` | JSONB | NULLABLE | state before the change; shape is `action`-specific and unvalidated (operational, not queried structurally) |
| `new_value` | JSONB | NULLABLE | state after the change; same shape rule as `old_value` |
| `metadata` | JSONB | NOT NULL, DEFAULT '{}' | `{ip_address?, user_agent?, admin_id?, reason?}` |
| `created_at` | TIMESTAMP | NOT NULL | UTC; immutable |

**Constraints:**

- `PRIMARY KEY (log_id)`
- FK `account_id → Account.id` (no cascade delete)
- FK `actor_account_id → Account.id` (no cascade delete)

**Indexes:**

- `(account_id, created_at)` — retrieve an account's audit history
- `created_at` — retention job scan

**Notes:**

- Written synchronously, in the same request/command handler as the change it records, via `AuditLogService.record(...)` (see `04-service-contracts.md`). It is never written via EventEmitter and never depends on event delivery — an audit write that fails must fail the request loudly (logged as an operational error) rather than silently drop the record, but it never blocks or reverses the already-committed business change it describes.
- Hard-deleted permanently 12 months after `created_at` by a retention job — see `10-background-jobs.md`. No archival tier exists in V1.
- Never exposed via any account-facing or public API. There is no admin UI in V1 scope; authorized access (Platform Operations, Security Personnel, authorized Support Staff) is direct, credentialed database access outside the application's public surface — the same operational access model already used for all other backend administration in V1.
- Excludes Social Accounts Setup entirely (sessions, Smart Mode input, Manual Mode input, normalization/URL-generation history are never logged) — only the resulting saved `ConnectedAccount` record is subject to normal audit rules.

**Logged Event Catalog (`action` values):**

```
account_created | account_deleted
authentication_method_added | authentication_method_removed | primary_identity_changed
connected_account_added | connected_account_updated | connected_account_removed | connected_account_reordered
display_name_updated | avatar_updated | bio_updated | contact_information_updated
admin_action | support_action | manual_intervention
```

`account_updated`, `recovery_email_changed`, `page_*`, and `block_*` values from the architectural policy are not logged in V1: `account.settings` mutations are not individually itemized (see `account_updated` scope note below), Recovery Email does not exist in V1 (`authentication.policy.v1.md`), there is no Page entity distinct from the single public profile, and Block create/update/delete/reorder are Product Data changes already fully covered by `Block` row history via `updated_at`/`sort_order`, not audit-logged individually — auditing scope in V1 is limited to identity, access, and account-lifecycle events, per the "Operational Purpose Only" principle in the architectural policy.

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

No other value is valid in V1. There is no `email`, `phone`, `password`, `facebook`, `x`, `linkedin`, `instagram`, `microsoft`, `github`, or `tiktok` provider value in V1. This enum is the V1 allowlist; a future version may extend it to support an additional Provider Adapter, but doing so is a configuration/allowlist change, not an architecture or route redesign (see `authentication.policy.v1.md` — "Provider Adapter Architecture").

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

### AnalysisSessionStatus

```
pending     — Analysis Session created; execution in progress
completed   — Provider execution succeeded; report/scores/suggestions/recommendations are populated
failed      — Provider execution failed or timed out; never reused
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
  ├─── AnalyticsEvent      (reference only, no FK enforcement)
  │
  ├─── AnalysisSession     (1:many, account_id FK, Hard Delete)
  │
  └─── AccountQRCode       (1:1, account_id UNIQUE FK, Hard Delete)

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
| Complete registration | `UsernameReservation` delete + `Account` create + `AuthenticationIdentity` create + `ProfileContent` create, executed in one database transaction. Immediately followed by a required blocking step (not optional, not deferred): SVG asset generation via `QRService.generate`, asset upload via `StoragePlatform.upload`, and `AccountQRCode` record creation (entity 12). The Account must not be treated as active until both the database transaction and this blocking QR creation step succeed; if QR creation fails, the registration must fail as a whole (the just-created Account row must not be left active without a QR Code record). |
| Block reorder | Multiple `Block` sort_order updates |
| Account deletion | `AccountService.deleteAccount` executes two steps: (1) `Account.status = 'deleted'`; (2) publishes `account.deleted` event. Sessions are invalidated synchronously via in-process EventEmitter handler (AuthService.revokeAllSessions). A second EventEmitter handler enqueues the `account.deletion.cascade` BullMQ job for durable cross-domain cleanup. The cascade job executes in order: (1) collect out_link_ids via OutLinksService.getAccountOutLinkIds (read-only, before any deletion); (2) AnalyticsEvents hard-deleted (profile_view by account_id; link_click by out_link_ids) — analytics deleted before OutLinks to guarantee idempotency on retry; (3) OutLinks hard-deleted; (4) ProfileContent hard-deleted (includes `appearance_config` JSONB field); (5) Blocks hard-deleted; (6) ImageAssets hard-deleted (StoragePlatform.delete per asset); (7) ConnectedAccounts hard-deleted; (8) AnalysisSessions hard-deleted (all statuses); (9) Binary avatar asset physically deleted; (10) `AccountQRCode` record hard-deleted and its `canonical_asset_id` SVG asset deleted via Storage Platform (AccountService.deleteQrCode). After this step, `/qr/{qr_code_id}` no longer resolves to an active profile. The Account record is retained permanently. `AccountQRCode` is a separate Account-owned entity (entity 12), not a field on Account. No AppearanceState entity (`appearance_config` is a JSONB field on ProfileContent, deleted with ProfileContent). No `AiDecision`, `AcceptedDecision`, or `SuggestionHistory` entity in V1 — AnalysisSession is the only AI-owned entity, deleted in step 8. Cascade order and idempotency rules: see `10-background-jobs.md` — Job 7. |

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
| AnalysisSessionRepository | AI Platform |
| AccountQRCodeRepository | Account |
