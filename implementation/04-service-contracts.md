# Minime V1 — Service Contracts

## Status

Canonical. Final.

---

## Architecture Authority

```
Architecture-Product-Definition/docs/MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md
Architecture-Product-Definition/account/account.model.specification.v1.md
Architecture-Product-Definition/account/authentication.policy.v1.md
Architecture-Product-Definition/account/username.policy.v1.md
Architecture-Product-Definition/account-management/connected.accounts.specification.v1.md
Architecture-Product-Definition/profile-content/profile.content.specification.v1.md
Architecture-Product-Definition/blocks/block.system.specification.v1.md
Architecture-Product-Definition/out-links/out-link.model.specification.v1.md
Architecture-Product-Definition/analytics/analytics.model.specification.v1.md
```

---

## Purpose

Defines the canonical backend service contracts for Minime V1.

Service contracts define:

- service ownership
- commands and queries
- allowed and forbidden dependencies
- emitted events

This document does not define HTTP endpoints.

This document does not define database schema.

---

## Global Service Rules

### Implementation

- Services must be NestJS injectable providers.
- Services must contain business use case coordination only.
- Services must access persistence only through owned repositories.
- Services may call Platform Services when required.
- Services may publish internal events after successful business operations.
- Services must not access Prisma directly.
- Controllers must not access Prisma directly.
- Repositories must return typed results.
- Services must not contain HTTP request or response logic.
- Services must not depend on controllers.
- Services must not own another Product Domain's business decisions.

### Commands

- Commands must validate input before mutation.
- Commands must persist changes through repositories.
- Commands must return the latest saved state.
- Commands must publish events only after successful persistence.

### Queries

- Queries must not mutate business state.
- Queries must apply ownership filters.
- Queries must exclude soft-deleted records by default.

### Cross-Service Dependencies

- Product Domain services may depend on another Product Domain service only when this document explicitly allows it.
- Platform Services may be used by any Product Domain service.
- Platform Services must not depend on Product Domain services.

### Service Lifetime

- Services must be stateless.
- Request-specific state must not be stored inside services.

---

## AccountService

**Owner:** Account domain

**Purpose:** Own account lifecycle, settings, and QR code configuration.

### Commands

```
createAccount({username, auth_type, auth_identifier})
  — creates Account + AuthCredential + ProfileContent in one database transaction
  — called by AuthService during registration completion; not a direct user-facing command

updateAccountSettings({account_id, settings})
  — mutates Account.settings

updateQrCodeConfiguration({account_id, qr_config})
  — mutates Account.qr_config (color, format fields)
  — clears Account.qr_config.storage_key to force QR regeneration on next request

generateQrCode({account_id})
  — internal command; not a direct user-facing HTTP endpoint
  — generates QR SVG for the account's canonical profile URL via StoragePlatform.upload(assetType: 'qr')
  — stores returned storage key in Account.qr_config.storage_key
  — called lazily when GET /api/v1/account/qr-code is requested and storage_key is null

deleteQrAsset({account_id})
  — internal command; called only by account.deletion.cascade job
  — if Account.qr_config.storage_key is set: calls StoragePlatform.delete(storage_key)
  — idempotent: safe to re-run

deleteAccount({account_id})
  — immediate and final; there is no grace period and no cancellation in V1
  — AccountService.deleteAccount executes: (1) Account.status = 'deleted'; (2) publishes account.deleted event; session revocation and cross-domain cascade are handled by EventEmitter handlers (see 06-event-contracts.md)
  — AccountService must not directly invoke repositories outside its allowed dependencies; cross-domain cleanup is delegated to the account.deletion.cascade BullMQ job
  — account record is retained permanently (account_id referenced by audit logs)
  — username permanently reserved; not released in V1
  — cascade order, idempotency, and partial failure rules: see 10-background-jobs.md — Job 7
```

### Queries

```
getAccountById({id})
getAccountByUsername({username})
getAccountSettings({account_id})
getQrCodeConfiguration({account_id})
```

### Allowed Dependencies

```
AccountRepository
ProfileContentRepository   — initialize ProfileContent row during createAccount
StoragePlatform            — QR code generation and deletion only
EventBus
```

### Forbidden Dependencies

```
AuthRepository
UsernameReservationRepository
BlockRepository
ConnectedAccountRepository
OutLinkRepository
AnalyticsRepository
AIService
```

### Emitted Events

```
account.created
account.settings.updated
account.qr.updated
account.deleted
```

### Implementation Rules

- AccountService must own account lifecycle.
- AccountService must own settings mutations.
- AccountService must own QR code configuration.
- `createAccount` must execute within a single database transaction (Account + AuthCredential + ProfileContent).
- AccountService must not mutate ProfileContent fields beyond creating the initial record.
- `deactivate` and `reactivate` operations do not exist in V1. `suspended` status is administrative and is not user-initiated.
- There is no `cancelAccountDeletion`. Deletion is immediate and final per architecture.
- `deleteAccount` sets Account.status = 'deleted' and publishes `account.deleted`. AccountService must not call repositories of other domains; cross-domain cascade executes via the `account.deletion.cascade` BullMQ job enqueued by an EventEmitter handler.
- Username is permanently reserved after deletion; it is not released in V1.

---

## AuthService

**Owner:** Authentication domain

**Purpose:** Own authentication flows, session lifecycle, and registration completion.

### Commands

```
verifyProviderAssertion({provider, provider_assertion})
  — validates provider assertion (e.g. Google or Apple ID token) against provider public keys
  — extracts provider_subject (sub claim) and provider_email from verified assertion
  — returns {account_id} if account exists, or {needs_registration: true, provider_assertion} if not

completeRegistrationWithProvider({reservation_id, provider, provider_assertion})
  — validates reservation exists
  — validates provider_assertion
  — calls AccountService.createAccount({username_from_reservation, provider, provider_subject, provider_email})
  — deletes UsernameReservation
  — creates AuthenticationIdentity record
  — creates Session; issues tokens

createSession({account_id})
  — creates Session record (stores refresh_token_hash)
  — issues short-lived access token (JWT, not stored)
  — issues refresh token (rotated on every use)

refreshSession({refresh_token})
  — validates refresh token against stored hash
  — rejects revoked or expired sessions
  — rotates refresh token (old hash invalidated, new hash stored)
  — issues new access token

revokeSession({session_id, account_id})
  — sets Session.revoked_at

revokeAllSessions({account_id})
  — sets revoked_at on all active, non-expired sessions for the account
```

### Queries

```
getCurrentSession({session_id})      — returns active Session
getActiveSessions({account_id})      — returns all active non-expired sessions
```

### Allowed Dependencies

```
AuthRepository              — Session, AuthenticationIdentity
AccountRepository           — read account during session creation
UsernameReservationRepository — read and delete reservation during registration
AccountService              — createAccount (registration only)
EventBus
```

### Forbidden Dependencies

```
ProfileContentRepository
BlockRepository
ConnectedAccountRepository
OutLinkRepository
AnalyticsRepository
StoragePlatform
AIService
```

### Emitted Events

```
auth.provider.authenticated
auth.registration.completed
auth.session.created
auth.session.refreshed
auth.session.revoked
auth.logout
```

### Implementation Rules

- Authentication methods, supported providers, and token security: See `08-security-model.md` and `07-validation-rules.md`.
- AuthService must not own profile content.
- AuthService must not implement Email OTP, direct email authentication, or AuthEmailDelivery.
- AuthService must not store provider access tokens or refresh tokens.

---

## UsernameService

**Owner:** Username domain

**Purpose:** Own username availability checking and reservation during registration.

### Commands

```
reserveUsername({username})
  — validates format (a-z/0-9, length 3–30, lowercase only)
  — checks availability against Account.username and active UsernameReservation
  — creates UsernameReservation record

expireReservation({id})
  — hard deletes expired UsernameReservation
  — called by background job only; no HTTP endpoint
```

### Queries

```
checkAvailability({username})
  — validates format
  — checks Account.username and active UsernameReservation
  — returns {username, available: boolean}
```

### Allowed Dependencies

```
UsernameReservationRepository
AccountRepository              — check if username already exists on Account
EventBus
```

### Forbidden Dependencies

```
AuthRepository
ProfileContentRepository
BlockRepository
ConnectedAccountRepository
AnalyticsRepository
StoragePlatform
AIService
```

### Emitted Events

```
username.reserved
username.reservation.expired
```

### Implementation Rules

- Username is set on Account.username during registration via AuthService. It is never changed in V1.
- There is no `changeUsername` command. Username changes are not supported in V1.
- There is no `assignUsername` as a standalone operation. Assignment happens in AuthService.completeRegistration.
- There is no `releaseUsername` as a user-facing operation. Reservations expire automatically via background job.
- Availability check must consult both Account.username (permanent) and UsernameReservation (temporary hold).

---

## SocialAccountsService

**Owner:** Social Accounts domain

**Purpose:** Processing domain. Validates platform handles and generates canonical URLs from platform rules. Owns no persistent storage.

### Commands

```
validateAndNormalize({platform, username})
  → validates platform is in approved SocialPlatform enum
  → validates username format against platform-specific rules
  → generates canonical URL from platform URL template
  → returns {platform, username, url} or throws validation error
```

### Queries

None. This service owns no persistent data.

### Allowed Dependencies

```
None — no repositories
```

### Forbidden Dependencies

```
ConnectedAccountRepository
AccountRepository
Any repository
EventBus
```

### Emitted Events

None. Processing domain emits no business events.

### Implementation Rules

- SocialAccountsService owns no persistent storage. No database calls.
- SocialAccountsService is an internal processing service called by ConnectedAccountsService.
- SocialAccountsService has no REST API.
- `verifySocialAccount` does not exist. Social account verification is not a V1 capability.
- Platform-specific rules (handle format, URL template) are embedded in application code as configuration, not in database.

---

## ConnectedAccountsService

**Owner:** Connected Accounts domain

**Purpose:** Own persistent storage of user-entered social links (ConnectedAccount records). Store, manage, and reorder.

### Commands

```
addConnectedAccount({account_id, platform, username})
  — calls SocialAccountsService.validateAndNormalize({platform, username})
  — persists ConnectedAccount{account_id, platform, username, url, sort_order}

updateConnectedAccount({id, account_id, username})
  — only username is editable; platform is immutable
  — calls SocialAccountsService.validateAndNormalize to regenerate url
  — persists updated ConnectedAccount{username, url}
  — finds all social_icons blocks owned by account_id whose content.accounts array references this connected_account_id
  — for each affected block: calls OutLinksService.archiveOutLink for every active OutLink associated with (block_id, connected_account_id)
  — for each affected block: calls OutLinksService.createOutLink with the new destination_url, block_id, block_type 'social_icons', and connected_account_id
  — after persistence: invalidates profile cache (profile:public:{username})

removeConnectedAccount({id, account_id})
  — ownership verified via account_id before deletion
  — before deletion: finds all social_icons blocks owned by account_id whose content.accounts array references this connected_account_id
  — before deletion: removes the connected_account_id entry from each affected block's content.accounts array (writes to BlockRepository)
  — before deletion: calls OutLinksService.archiveOutLink for any active OutLinks associated with (block_id, connected_account_id) for each affected block
  — then: Hard Delete of ConnectedAccount; no soft delete, no archive, no trash
  — after deletion: invalidates profile cache

reorderConnectedAccounts({account_id, ordered_ids})
  — updates sort_order for each ConnectedAccount in one transaction

deleteAccountConnectedAccounts({account_id})
  — internal operation; called only by account.deletion.cascade job, not by controllers or HTTP requests
  — hard-deletes all ConnectedAccount records for the account
  — idempotent: safe to re-run
```

### Queries

```
getConnectedAccounts({account_id})     — list ordered by sort_order
getConnectedAccount({id, account_id})  — single record with ownership check
```

### Allowed Dependencies

```
ConnectedAccountRepository
SocialAccountsService          — validation and URL generation only
BlockRepository                — read and update social_icons block content during connected account removal
OutLinksService                — archive OutLinks for removed social icons
CacheDataService (Redis)       — cache invalidation after connected account removal
EventBus
```

### Forbidden Dependencies

```
AuthRepository
AccountRepository
ProfileContentRepository
AnalyticsRepository
StoragePlatform
AIService
```

### Emitted Events

```
connected-account.added
connected-account.updated
connected-account.removed
connected-account.reordered
```

### Implementation Rules

- ConnectedAccountsService stores lightweight social links. Not OAuth connections.
- ConnectedAccountsService must not store OAuth tokens, provider metadata, or access credentials.
- Deletion is Hard Delete. ConnectedAccount has no deleted_at, no archive, no status lifecycle.
- platform is not editable. Platform changes require delete + create.
- url is always system-generated from platform rules. Users never provide URLs directly.
- There is no `connectProvider`, `disconnectProvider`, `refreshProviderConnection`, or `revokeProviderConnection`.
- There is no `verifySocialAccount` or `connectionStatus`.

---

## ProfileService

**Owner:** Profile domain

**Purpose:** Own profile content (display name, bio, avatar, contact, appearance) and block lifecycle.

### Commands

```
updateProfileContent({account_id, display_name?, bio?, contact?})
  — updates ProfileContent fields; validates fields per `07-validation-rules.md`

uploadAvatar({account_id, file})
  — stores file via StoragePlatform
  — updates ProfileContent.avatar_storage_key with returned storage key

addBlock({account_id, type, content, settings, sort_order, style_overrides?})
  — validates type against approved BlockType enum
  — validates content and settings against per-type schema
  — enforces max 1 per account for identity blocks (avatar, name, bio)
  — enforces MAX_BLOCKS_PER_ACCOUNT total active blocks per account; rejects with business error if limit is reached
  — creates Block record; style_overrides stores only explicit user-authored style keys; absent or null means full inheritance from the active Theme at render time
  — if block type is `button`: calls OutLinksService.createOutLink once (connected_account_id = null)
  — if block type is `social_icons`: calls OutLinksService.createOutLink once per entry in content.accounts (connected_account_id = that entry's connected_account_id)

updateBlock({id, account_id, content?, settings?, style_overrides?})
  — validates ownership via account_id
  — validates updated content/settings against per-type schema
  — mutates Block.content and/or Block.settings and/or Block.style_overrides
  — style_overrides stores only explicit user-authored style keys; null clears all overrides (full inheritance from active Theme); inherited values must not be stored
  — for `button` blocks: if label or url changes, archives old OutLink and creates new OutLink
  — for `social_icons` blocks: compares old and new content.accounts; archives OutLinks for removed entries; creates OutLinks for added entries; entries with unchanged connected_account_id retain their OutLink

deleteBlock({id, account_id})
  — validates ownership via account_id
  — soft deletes Block (sets deleted_at)
  — archives all active OutLinks for the block (one for button; one per icon for social_icons)

reorderBlocks({account_id, ordered_ids})
  — validates all block IDs belong to account_id
  — updates Block.sort_order for all listed blocks in one transaction

updateAppearance({account_id, selected_theme_id?, customizations?})
  — updates ProfileContent.appearance_config
  — validates selected_theme_id against the theme catalog at `packages/config/themes.ts`; rejects unknown theme IDs

uploadImage({account_id, file})
  — validates file MIME type and file size per 07-validation-rules.md — ImageAsset constants
  — uploads file via StoragePlatform (assetType: 'image'); receives back storage_key and processed dimensions
  — creates ImageAsset record {account_id, storage_key, mime_type, width, height, file_size}
  — returns {image_id, width, height, file_size, mime_type} for use in image block content
  — storage key is never returned to callers beyond this service

deleteImage({id, account_id})
  — validates ownership via account_id
  — validates no active (non-soft-deleted) Block for this account has content->>'image_id' = id
  — if referenced by an active block: rejects with business error (user must delete the block first)
  — hard-deletes ImageAsset record
  — calls StoragePlatform.delete(storage_key)

deleteAccountData({account_id})
  — internal operation; called only by account.deletion.cascade job, not by controllers or HTTP requests
  — hard-deletes all Block records for the account
  — hard-deletes all ImageAsset records for the account; calls StoragePlatform.delete(storage_key) for each
  — hard-deletes ProfileContent record for the account (includes appearance_config JSONB field)
  — calls StoragePlatform.delete(avatar_storage_key) if avatar_storage_key is set (binary avatar asset cleanup)
  — idempotent: safe to re-run
```

### Queries

```
getProfileContent({account_id})    — returns ProfileContent record
getBlocks({account_id})            — returns active Blocks ordered by sort_order
getBlock({id, account_id})         — returns one Block with ownership check
```

### Allowed Dependencies

```
ProfileContentRepository
BlockRepository
ImageAssetRepository  — uploadImage, deleteImage, deleteAccountData
OutLinksService       — OutLink creation and archiving triggered by block lifecycle events
StoragePlatform       — avatar upload/deletion and image asset upload/deletion
EventBus
```

### Forbidden Dependencies

```
AuthRepository
UsernameReservationRepository
ConnectedAccountRepository
OutLinkRepository
AnalyticsRepository
AIService
```

### Emitted Events

```
profile.updated
profile.avatar.updated
profile.image.uploaded
profile.image.deleted
profile.block.added
profile.block.updated
profile.block.deleted
profile.block.reordered
profile.appearance.updated
```

### Implementation Rules

- ProfileService must not implement `publishProfile` or `unpublishProfile`. There is no publishing workflow in V1.
- ProfileService does not have a `createProfile` command. ProfileContent is created by AccountService during registration.
- Saving any profile change makes it publicly visible within the cache TTL (≤ 60 seconds). There is no separate "publish" step.
- All blocks reference account_id. There is no profile_id.
- Identity blocks (avatar, name, bio) are limited to max 1 per account. This limit must be enforced on addBlock.
- Content blocks (image, social_icons, button, divider, title, textbox) are unlimited.
- avatar_storage_key stores the Storage Platform key only. Binary data lives in Object Storage.
- ImageAsset storage keys must never be returned in API responses. Public image URLs are constructed at render time via StoragePlatform.buildPublicUrl.
- An image block's content.image_id must reference an ImageAsset owned by the same account. This is validated on addBlock and updateBlock.
- `uploadImage` must be called before creating or updating an image block with a new image. The returned image_id is then used in the block content.
- `deleteImage` rejects if any active block references the image. The user must delete the block first.

---

## RenderingService

**Owner:** Rendering domain

**Purpose:** Produce renderable public profile data from canonical business state. Stateless. Never writes.

### Commands

```
invalidateCache({account_id})
  — removes cached rendered profile from Redis
```

### Queries

```
getRenderedProfile({username})
  — resolves username to Account
  — reads ProfileContent, Blocks, ConnectedAccounts, OutLinks, ImageAssets (all read-only)
  — returns cached result if fresh (TTL ≤ 60 seconds); builds from database if stale
```

### Allowed Dependencies

```
AccountRepository           — read-only
ProfileContentRepository    — read-only
BlockRepository             — read-only
ConnectedAccountRepository  — read-only
OutLinkRepository           — read-only; resolves active OutLink public_ids for tracked clickable blocks
ImageAssetRepository        — read-only; resolves ImageAsset storage keys to public URLs for image blocks
StoragePlatform             — read-only; buildPublicUrl only; RenderingService never uploads or deletes
CacheDataService (Redis)
```

### Allowed Dependencies (additional)

```
EventBus   — for emitting profile.viewed after successful public profile presentation
```

### Forbidden Dependencies

```
AuthRepository
AnalyticsRepository
Any write operation
```

### Emitted Events

```
profile.viewed
```

### Implementation Rules

- RenderingService must be stateless.
- RenderingService must never write to any repository.
- RenderingService reads directly from repositories for performance. It does not go through other domain services.
- After `getRenderedProfile` returns successfully and the public profile response is delivered, the public profile rendering pipeline must determine `device_category` using the platform device classification policy before emitting `profile.viewed`. Supported values: `desktop`, `mobile`, `tablet`. Device classification must occur in the render path before the event is emitted; Analytics must not classify, recalculate, or infer device_category.
- `profile.viewed` must be emitted only after a successful public profile presentation. It must not be emitted on cache hit failures, rendering errors, or suspended/deleted account responses.
- Public rendering must not wait on `profile.viewed` emission or on analytics persistence. If event emission fails, profile rendering must still succeed.
- For `button` blocks: look up one active OutLink by `(block_id, connected_account_id = null)` via OutLinkRepository. Render as `/out/{public_id}`.
- For `social_icons` blocks: look up one active OutLink per icon by `(block_id, connected_account_id)` via OutLinkRepository. Each icon in the rendered output uses its own `/out/{public_id}`. Icons with no active OutLink are omitted from the rendered output.
- Raw destination URLs from block content must not appear in public render payloads for tracked links.
- If no active OutLink exists for a button block or for a specific social icon, that link is omitted. It must not fall back to the raw URL.
- RenderingService uses the same theme catalog at `packages/config/themes.ts` to resolve theme values from `selected_theme_id`.
- For `image` blocks, RenderingService must look up the `ImageAsset` by `content.image_id` via ImageAssetRepository and call `StoragePlatform.buildPublicUrl(storage_key)` to produce the public image URL. The storage key must not appear in the rendered payload. If the referenced ImageAsset does not exist, the block must be omitted or rendered as a placeholder.
- Cache TTL and cache keys: See `09-caching-strategy.md`.
- There is no publication state. Rendering always reflects the current persisted state of profile content.

---

## OutLinksService

**Owner:** Out Links domain

**Purpose:** Own outbound link records, lifecycle, and click tracking initiation.

**Creation model:** OutLinks are created internally by the system. They are NOT created via user-facing API. An OutLink is created whenever a new Clickable State is created — i.e., when a Button Block or Social Icons Block is added, or when a change to label, URL, or platform creates a new Clickable State. ProfileService calls OutLinksService to create and archive OutLinks as part of block lifecycle operations.

### Commands

```
createOutLink({account_id, block_id, block_type, destination_url, label_snapshot, source_snapshot, connected_account_id})
  — internal operation; called by ProfileService, not by controllers
  — connected_account_id: null for button blocks; ConnectedAccount ID for social_icons blocks
  — triggered when a new Clickable State is created (new button block, or new social icon entry)
  — generates unique URL-safe public_id per `07-validation-rules.md` — Out Links constants
  — creates OutLink with status 'active'

archiveOutLink({id, account_id})
  — internal operation; called by ProfileService or ConnectedAccountsService, not by controllers
  — triggered when a Clickable State changes or is removed (label/url/platform change, social icon removed, block deleted)
  — validates ownership via account_id
  — sets OutLink.status = 'archived'
  — sets OutLink.archived_at = NOW()

resolveOutLink({public_id})
  — called by the public redirect route GET /out/{public_id}
  — looks up OutLink by public_id
  — returns destination_url for redirect
  — enqueues BullMQ job to record click event (non-blocking)

purgeArchivedOutLinks()
  — background job only
  — hard deletes records where status = 'archived' AND archived_at < NOW() - 90 days

getAccountOutLinkIds({account_id})
  — internal read-only query; called only by account.deletion.cascade job before OutLink deletion
  — returns all OutLink IDs (any status) owned by account_id
  — used to collect out_link_ids before deletion so analytics cleanup does not depend on delete return values
  — idempotent: safe to call multiple times; returns empty list if OutLinks already deleted

deleteAccountOutLinks({account_id})
  — internal operation; called only by account.deletion.cascade job, not by controllers or HTTP requests
  — hard-deletes all OutLink records for the account regardless of status
  — idempotent: safe to re-run
```

### Queries

```
getOutLinks({account_id})           — returns OutLinks with status != 'archived' for account
getOutLink({id, account_id})        — returns one OutLink with ownership check
```

### Allowed Dependencies

```
OutLinkRepository
EventBus
```

### Forbidden Dependencies

```
AuthRepository
AccountRepository
ProfileContentRepository
BlockRepository
ConnectedAccountRepository
AnalyticsRepository
StoragePlatform
AIService
```

### Emitted Events

```
out-link.created
out-link.archived
out-link.clicked
```

### Implementation Rules

- OutLink has no `updateOutLink` command. All content fields (destination_url, label_snapshot, source_snapshot, block_id, block_type, public_id) are immutable after creation.
- OutLink has no `enableOutLink` or `disableOutLink`. There is no is_enabled field.
- OutLink has no `deleteOutLink`. Lifecycle is: active → archived → hard purge after 90 days.
- OutLinks are not created via user-facing API. They are created internally by the system (via ProfileService calling OutLinksService) when a new Clickable State is created.
- Archiving is not user-initiated. It is triggered internally when a Clickable State changes or a block is deleted.
- Click tracking must not block redirect. `resolveOutLink` returns destination_url immediately; analytics recording is async via BullMQ.
- public_id format and constraints: See `07-validation-rules.md` — Out Links constants. Must be globally unique.

---

## AnalyticsService

**Owner:** Analytics domain

**Purpose:** Own analytics event ingestion and aggregation. Observational only. Never modifies business entities.

### Commands

```
recordProfileView({account_id, device_category})
  — appends AnalyticsEvent{event_type: 'profile_view', account_id, device_category, created_at}

recordLinkClick({out_link_id})
  — appends AnalyticsEvent{event_type: 'link_click', out_link_id, created_at}

purgeOldEvents()
  — background job only
  — hard deletes AnalyticsEvent records past retention period

deleteAccountAnalytics({account_id, out_link_ids})
  — internal operation; called only by account.deletion.cascade job, not by controllers or HTTP requests
  — hard-deletes AnalyticsEvent records where event_type = 'profile_view' AND account_id = account_id
  — hard-deletes AnalyticsEvent records where event_type = 'link_click' AND out_link_id IN (out_link_ids)
  — out_link_ids are collected by the cascade job via OutLinksService.getAccountOutLinkIds BEFORE OutLinks are deleted; analytics runs before OutLink deletion to ensure idempotency on retry
  — accepts empty out_link_ids list gracefully (only profile_view deletion runs)
  — idempotent: safe to re-run
```

### Queries

```
getProfileAnalytics({account_id, from, to})
  — aggregated profile_view data: view count, device_category breakdown

getOutLinkAnalytics({out_link_id, from, to})
  — aggregated link_click data: click count

getAnalyticsSummary({account_id})
  — summary: total profile views, total out link clicks
```

### Allowed Dependencies

```
AnalyticsEventRepository
EventBus
```

### Forbidden Dependencies

```
AccountRepository
AuthRepository
ProfileContentRepository
BlockRepository
ConnectedAccountRepository
OutLinkRepository
StoragePlatform
AIService
```

### Emitted Events

```
analytics.retention.completed
```

### Implementation Rules

- AnalyticsService must never modify business entities.
- Valid event types: See `03-canonical-data-model.md` — AnalyticsEventType enum.
- `recordProfileView` accepts `{account_id, device_category}` as provided in the `profile.viewed` event payload emitted by the public profile rendering pipeline. AnalyticsService must not classify, recalculate, or infer `device_category`. The value received is the sole value recorded.
- No visitor identity stored (no username, no IP, no session).
- No geographic data stored (no country).
- No referrer data stored.
- Aggregation reports derive from `profile_view` and `link_click` events only.

---

## Platform Service Usage

| Platform Service | Allowed Usage |
|---|---|
| Data (PostgreSQL via Prisma) | Accessed through repositories only; never from controllers or services directly |
| Storage (Object Storage) | File upload, key storage, and URL construction; ProfileService (avatar and image asset upload/deletion); RenderingService (buildPublicUrl only — read-only) |
| Events (NestJS EventEmitter) | In-process event notification after business persistence |
| AI | On-demand Analysis Session via `AIService.analyzeProfile`; triggered only by explicit "Analyze My Profile" user request; AI responses require user confirmation before any business mutation |

**NestJS EventEmitter characteristics:**

- In-process delivery (same Node.js instance only).
- No guaranteed delivery across restarts.
- No automatic retry on handler failure.
- No distributed or cross-process delivery.
- For durable async work requiring retry, use BullMQ jobs.

---

## Dependency Direction

```
Controller
    ↓
Service
    ↓
Repository
    ↓
Database

Service also connects to:
    ├──► Platform Services (Storage, AI)
    └──► Event Bus (NestJS EventEmitter)
              └──► BullMQ Jobs (durable async work)
```

Reverse dependencies are prohibited.

---

## Repository Ownership

| Repository | Owning Domain |
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

Services must access only repositories owned by their domain.

Shared reads must execute through approved service contracts.

---

## Service Prohibitions

Implementation must not:

- place business logic inside controllers
- access Prisma outside repositories
- expose repositories to controllers
- allow repositories to publish events
- allow repositories to call Platform Services
- allow Platform Services to own Product Domain decisions
- allow services to mutate another Product Domain through its repository
- introduce a User, Profile, PublicProfile, or SocialAccount entity or repository
- introduce a NotificationService, notification inbox, notification preferences, push notification system, or any product notification system — Minime V1 has no email delivery capability; authentication is provider-based
- reference profile_id anywhere
- implement publishProfile, unpublishProfile, or any profile publication command
- implement changeUsername (username is immutable in V1)
- implement Email OTP, phone OTP, SMS authentication, or AuthEmailDelivery
- implement OAuth provider connections (connectProvider, refreshProviderConnection, revokeProviderConnection)
- implement social account verification (verifySocialAccount)
- claim distributed event delivery through NestJS EventEmitter
