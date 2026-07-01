# Minime V1 — Implementation Plan

## Status

Canonical. Final.

---

## Architecture Authority

```
All implementation documents 01–12
Architecture-Product-Definition/docs/MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md
Architecture-Product-Definition/platform/platform.architecture.specification.v1.md
```

---

## Purpose

Defines the phased execution sequence for implementing Minime V1.

This plan sequences implementation to respect dependency order: each phase depends only on phases already complete. It does not prescribe internal implementation details, sprint assignments, or team allocation — those are operational decisions.

The plan covers: infrastructure, product domains (all 9), platform services (all 4), and cross-cutting capabilities (SEO, Integrations).

---

## Dependency Order

```
01 technology stack
02 repository structure
    │
    ▼
03 canonical data model (entities, migrations)
    │
    ▼
04 service contracts
    │
    ├── 05 api contracts
    ├── 06 event contracts
    ├── 07 validation rules
    ├── 08 security model
    ├── 09 caching strategy
    ├── 10 background jobs
    ├── 11 platform services (storage, ai)
    └── 12 seo and integrations
```

---

## Phase 1 — Project Bootstrap

**Reference:** `01-technology-stack.md`, `02-repository-structure.md`

**Deliverables:**

- Turborepo monorepo initialized with `apps/web`, `apps/api`, `packages/shared`, `packages/validation`, `packages/config`, `packages/ui`
- NestJS + Fastify application scaffolded in `apps/api`
- Next.js App Router application scaffolded in `apps/web`
- PostgreSQL connection configured and verified
- Redis connection configured and verified
- Object Storage client configured (Minio for development, provider-specific for production)
- Pino logger configured
- Vitest configured for unit tests
- Playwright configured for end-to-end tests
- GitHub Actions CI pipeline: lint, type-check, test on every pull request
- Docker Compose for local development (PostgreSQL, Redis, Minio, API, Web)
- Environment variable structure aligned with `08-security-model.md` — Secret Management

**Complete when:** `apps/api` and `apps/web` start without errors; all external services connect.

---

## Phase 2 — Data Platform

**Reference:** `03-canonical-data-model.md`

**Deliverables:**

- All 14 Prisma models defined exactly as specified in `03-canonical-data-model.md`:
  `Account`, `AuthenticationIdentity`, `Session`, `UsernameReservation`, `ProfileContent`, `Block`, `ImageAsset`, `ConnectedAccount`, `OutLink`, `AnalyticsEvent`, `AnalysisSession`, `AccountQRCode`, `AccountDeletionOutbox`, `AuditLog`
- All enum types: `AccountStatus`, `AuthProvider`, `BlockType`, `SocialPlatform`, `AnalyticsEventType`, `DeviceCategory`, `OutLinkStatus`, `AnalysisSessionStatus`
- All indexes and uniqueness constraints as specified, including `PRIMARY KEY(qr_code_id)` and `UNIQUE(account_id)` on `AccountQRCode`; the partial unique indexes on `Block` (`(account_id, type) WHERE type IN ('avatar','name','bio') AND deleted_at IS NULL`, and `(account_id, sort_order) WHERE deleted_at IS NULL`), `ConnectedAccount` (`(account_id, sort_order)`), and `OutLink` (`(block_id, connected_account_id) WHERE status = 'active'`)
- All repositories scaffolded (empty implementations, correct interfaces):
  `AccountRepository`, `AuthRepository`, `UsernameReservationRepository`, `ProfileContentRepository`, `BlockRepository`, `ImageAssetRepository`, `ConnectedAccountRepository`, `OutLinkRepository`, `AnalyticsEventRepository`, `AnalysisSessionRepository`, `AccountQRCodeRepository`, `AccountDeletionOutboxRepository`, `AuditLogRepository`
- Initial Prisma migration applied to development database

**Forbidden (per `03-canonical-data-model.md`):** No `User`, `Profile`, `PublicProfile`, or `SocialAccount` models.

**Complete when:** `prisma migrate dev` succeeds; all repositories injectable without errors.

---

## Phase 3 — Authentication System

**Reference:** `03-canonical-data-model.md`, `04-service-contracts.md`, `05-api-contracts.md`, `06-event-contracts.md`, `07-validation-rules.md`, `08-security-model.md`, `10-background-jobs.md`

**Deliverables:**

- `AuthenticationProviderAdapter` contract implemented; `GoogleAuthenticationProviderAdapter` and `AppleAuthenticationProviderAdapter` implement it; `resolveProviderAdapter(provider)` dispatches to the correct adapter and rejects any value outside the V1 allowlist
- `AuthService` implemented: `startOAuth`, `handleOAuthCallback`, `completeRegistrationWithOAuth`, `linkAuthenticationProvider`, `unlinkAuthenticationProvider`, `refreshOAuthIdentityMetadata`, `createSession`, `refreshSession`, `revokeSession`, `revokeAllSessions` — none of these contain Google-specific or Apple-specific branching beyond `resolveProviderAdapter`
- `provider` constrained to `google` | `apple` at every method and endpoint boundary in V1; no generic open-ended provider value; `provider` is always request data, never a route/path segment
- Google adapter: backend-driven OAuth authorization-code exchange; `id_token` verified against Google's public keys (signature, issuer, audience, expiry); `sub` claim normalized to `provider_subject`; `email`/`email_verified`/name/avatar normalized into `email`/`email_verified`/`provider_profile`
- Apple adapter: backend-driven OAuth authorization-code exchange using a Services ID `client_id` and a server-generated JWT `client_secret` signed with the Apple private key; `id_token` verified against Apple's public keys (signature, issuer, audience, expiry); `sub` claim normalized to `provider_subject`; one-time `name`/`email` payload captured on first authorization into `email`/`provider_profile`; relay email stored identically to a direct email; `response_mode=form_post` callback delivery normalized to the same `{provider, code, state}` shape before the core flow runs
- `state` and `nonce` generated at `startOAuth`, validated at `handleOAuthCallback` (provider-agnostic core controls, not adapter logic); both single-use; callback validates `provider` from request data against the provider bound to the stored state context, rejecting mismatches
- `oauth_handoff_token` issued after a validated callback with no matching account (short-lived, single-use, used for registration handoff); `link_handoff_token` issued after a validated callback for `intent=link` with no existing same-account identity (short-lived, single-use, used to finalize linking); neither ever contains raw provider tokens
- JWT access token issuance (short-lived, not stored)
- Refresh token rotation (hash stored in Session, plain text discarded after issuance)
- `AuthRepository` implemented: Session CRUD, AuthenticationIdentity CRUD (fields: `auth_identity_id`, `account_id`, `provider`, `provider_subject`, `email`, `email_verified`, `provider_profile`, `created_at`, `updated_at` — no `display_name` column)
- Duplicate-identity detection by `(provider, provider_subject)` enforced at registration, sign-in, and linking; `email` never consulted for this lookup; no automatic account merge implemented anywhere
- Last-remaining-provider protection enforced: `unlinkAuthenticationProvider` rejects removal of an account's only `AuthenticationIdentity`, addressed by `auth_identity_id`
- API endpoints implemented (all provider-agnostic; `provider` is always request data): `POST /auth/oauth/start`, `POST /auth/oauth/callback`, `POST /auth/oauth/register`, `GET /account/auth-providers`, `POST /account/auth-providers/link`, `DELETE /account/auth-providers/{auth_identity_id}`, `POST /auth/session/refresh`, `POST /auth/logout`, `POST /auth/logout-all`, `GET /auth/sessions`
- Scheduled job: `auth.session.cleanup`
- Auth guard (JWT validation middleware)
- Events emitted: `auth.provider.started`, `auth.provider.completed`, `auth.provider.failed`, `auth.provider.linked`, `auth.provider.unlinked`, `auth.registration.completed`, `auth.session.refreshed`
- No Email OTP, phone OTP, SMS, WhatsApp, password, Facebook Login, X Login, or LinkedIn Login implemented anywhere in this phase
- No provider-specific route segments anywhere in this phase (no `/auth/{provider}/*`, no `/auth/google/*`, no `/auth/apple/*`)

**Security requirements (per `08-security-model.md`):**
- `state` validated for CSRF protection; `nonce` validated for replay protection; both single-use; both provider-agnostic core controls
- Callback validates `provider` (request data) against the provider bound to the stored state context; rejects mismatch; rejects unsupported providers before resolving an adapter
- Token exchange (`code` → tokens) executed by the backend only, inside the resolved adapter; no client secret ever reaches the frontend
- `id_token` validated server-side: signature, issuer, audience, expiry, and nonce
- OAuth provider secrets stored per adapter (Google: `client_id`/`client_secret`; Apple: Services ID, Team ID, Key ID, private key); no adapter secret exposed to frontend; core `AuthService` never holds a provider secret directly
- `oauth_handoff_token` and `link_handoff_token` never stored in the database or Redis beyond their TTL; never logged
- Refresh tokens never stored plain-text
- Provider access/refresh tokens never persisted
- `email` never used as the canonical identity key, uniqueness constraint, or automatic-merge signal for any provider
- `primary_provider` never consulted for ownership/authorization; changing it never affects authentication validity

**Complete when:** Google Sign-In and Apple Sign-In OAuth flows (start → callback → registration or sign-in) succeed end-to-end through the provider-agnostic routes and adapter dispatch, with full state/nonce/token validation; registration via either provider completes with AuthenticationIdentity created; linking (via the handoff-token finalize step) and unlinking (by `auth_identity_id`) a provider both work, with the last-remaining-provider protection enforced; JWT guard rejects invalid tokens; adding a third adapter in a follow-up change requires no route, schema, or core service signature changes.

---

## Phase 4 — Account and Username

**Reference:** `03-canonical-data-model.md`, `04-service-contracts.md`, `05-api-contracts.md`, `06-event-contracts.md`, `07-validation-rules.md`, `08-security-model.md`

**Deliverables:**

- `AccountService` implemented: `createAccount({account_id, username})` (creates Account + ProfileContent only — see `04-service-contracts.md` — AuthService "Registration Transaction Coordination" for why AccountService never creates AuthenticationIdentity), `deleteAccount`, `getAccount`, `createQrCode`, `getQrCode`, `resolveQrCode`, `regenerateQrAsset`, `deleteQrCode`
- `AccountService.deleteAccount` implemented: in one transaction, sets Account.status = 'deleted' and inserts an `AccountDeletionOutbox` row; after commit, writes the `account_deleted` AuditLog row and publishes `account.deleted` (best-effort). The durable guarantee that `account.deletion.cascade` gets enqueued comes from the Deletion Outbox Dispatcher (Job 8) reading `AccountDeletionOutbox`, not from the EventEmitter publish — see `04-service-contracts.md` and `10-background-jobs.md` — Job 8
- `AccountDeletionOutboxRepository` implemented; Deletion Outbox Dispatcher (Job 8, scheduled every 30s) implemented in `apps/worker`
- `QRService` implemented: internal QR SVG generation utility using an internal QR library (no external provider, no provider abstraction)
- `AccountQRCodeRepository` implemented
- `AccountService.createQrCode` implemented: called exactly once, as a required blocking step immediately after `createAccount` establishes the canonical public profile route and before the Account becomes active; generates `qr_code_id`, builds `qr_url = https://minime.ae/qr/{qr_code_id}`, calls `QRService.generate`, stores the SVG via `StoragePlatform.upload(assetType: 'qr')`, persists one `AccountQRCode` record (`PRIMARY KEY(qr_code_id)`, `UNIQUE(account_id)`) — no `username` or username snapshot is stored
- Account activation gate enforced: if `createQrCode` fails (record persistence or SVG asset generation/persistence), the Account must not become active and registration must fail as a whole
- `AccountService.resolveQrCode` implemented: backs the public `GET /qr/{qr_code_id}` redirect; resolves `account_id`, then reads `Account.username` live through `account_id` (never from the QR Code record), then resolves the current public profile route, or returns not-found
- `AccountService.regenerateQrAsset` implemented: system-triggered only (missing/corrupted asset recovery, storage migration, format migration); preserves `qr_code_id`, `account_id`, `qr_url`; never user-facing
- There is no `updateQrCodeConfiguration`, no QR customization, no user-facing QR regeneration endpoint, and no lazy QR generation triggered by dashboard, settings, download, share, or public profile views
- `account.deletion.cascade` job scaffolded in this phase; fully implemented with all domain service calls in Phase 6 (after OutLinksService, AnalyticsService, ProfileService, and ConnectedAccountsService deletion commands are available)
- `UsernameService` implemented: `checkAvailability`, `reserveUsername`, `expireReservation`
- `UsernameReservationRepository` implemented
- Scheduled job: `username.reservation.expire`
- API endpoints implemented: `GET /usernames/availability`, `POST /usernames/reservations`, `GET /account`, `PATCH /account/settings`, `GET /account/qr-code`, `POST /account/deletion`
- Public redirect endpoint implemented: `GET /qr/{qr_code_id}` (302 to `/{username}`, or 404)
- Events emitted: `account.created`, `account.deleted`, `account.qr.created`, `username.reserved`
- `AuditLogRepository` and `AuditLogService` implemented (write-only `record` command; see `04-service-contracts.md` — AuditLogService); `AccountService.createAccount`/`deleteAccount` and `AuthService.linkAuthenticationProvider`/`unlinkAuthenticationProvider` call it; Audit Log Retention (Job 9, scheduled daily) implemented in `apps/worker`
- `UsernameService.reserveUsername` and `AuthService.completeRegistrationWithOAuth` both acquire the shared per-username Postgres advisory lock described in `04-service-contracts.md` — UsernameService.reserveUsername

**Invariants:**
- Username immutable after registration
- Account deletion is immediate, final, and irreversible
- Username permanently reserved after account deletion
- Exactly one `AccountQRCode` record exists per Account (`UNIQUE(account_id)`), and exactly one Account references each `AccountQRCode` record (`PRIMARY KEY(qr_code_id)`); created once, never user-editable
- No Account may reach `active` status without a successfully created and persisted `AccountQRCode` record and canonical SVG asset

**Complete when:** Registration flow completes (username → account created → QR Code created and persisted → Account becomes active); account deletion cascade executes correctly and completely.

---

## Phase 5 — Profile, Blocks, and Connected Accounts

**Reference:** `03-canonical-data-model.md`, `04-service-contracts.md`, `05-api-contracts.md`, `06-event-contracts.md`, `07-validation-rules.md`, `11-platform-services.md`

**Deliverables:**

- `ProfileService` implemented: `getProfileContent`, `updateProfileContent`, `uploadAvatar`, `uploadImage`, `deleteImage`, `addBlock`, `updateBlock`, `reorderBlocks`, `removeBlock`
- `ProfileContentRepository`, `BlockRepository`, `ImageAssetRepository` implemented
- Identity block instance limit enforced (max 1 per account for `avatar`, `name`, `bio`)
- MAX_BLOCKS_PER_ACCOUNT enforced: block creation rejected when total active block count reaches limit (canonical definition and default in `03-canonical-data-model.md`)
- Theme catalog implemented at `packages/config/themes.ts`; `updateAppearance` validates `selected_theme_id` against this catalog; `RenderingService` resolves theme values from the same catalog
- `ConnectedAccountsService.removeConnectedAccount` cleans up social_icons block references before hard-deleting: removes connected_account_id from all affected block content arrays, archives associated OutLinks
- Avatar upload: MIME validation → WebP conversion → `StoragePlatform.upload` → key stored in `ProfileContent.avatar_storage_key`
- Image block upload: MIME validation → WebP conversion → `StoragePlatform.upload` → `ImageAsset` record created → `image_id` returned to caller
- `image` block content validated: `image_id` must reference an `ImageAsset` owned by the same account
- `StoragePlatform` service implemented: `upload`, `delete`, `buildPublicUrl`
- `SocialAccountsService` implemented: URL generation from `{platform, username}` per per-platform rules in `Architecture-Product-Definition/social-accounts/social-platforms/`
- `ConnectedAccountsService` implemented: `addConnectedAccount`, `updateConnectedAccount`, `removeConnectedAccount`, `reorderConnectedAccounts`
- `ConnectedAccountRepository` implemented (Hard Delete)
- API endpoints implemented: `GET /profile`, `PATCH /profile`, `POST /profile/avatar`, `POST /profile/images`, `DELETE /profile/images/:imageId`, `GET /profile/blocks`, `POST /profile/blocks`, `PATCH /profile/blocks/:id`, `PATCH /profile/blocks/reorder`, `DELETE /profile/blocks/:id`, `GET /account/connected-accounts`, `POST /account/connected-accounts`, `PATCH /account/connected-accounts/:id`, `DELETE /account/connected-accounts/:id`, `PATCH /account/connected-accounts/reorder`
- Events emitted: `profile.updated`, `profile.image.uploaded`, `profile.image.deleted`, `profile.block.added`, `profile.block.updated`, `profile.block.deleted`, `connected-account.added`, `connected-account.removed`

**OutLink creation side-effect:** When a `button` or `social_icons` block is created, `ProfileService` calls `OutLinksService.createOutLink` internally. See Phase 6.

**Complete when:** Profile content, blocks, and connected accounts are fully manageable; avatar and image block uploads produce WebP assets with public URLs; image block reference validation enforces account ownership.

---

## Phase 6 — Out Links and Analytics

**Reference:** `03-canonical-data-model.md`, `04-service-contracts.md`, `05-api-contracts.md`, `06-event-contracts.md`, `10-background-jobs.md`

**Deliverables:**

- `OutLinksService` implemented: `createOutLink`, `archiveOutLink`, `resolveOutLink`
- `OutLinkRepository` implemented
- OutLink public_id generation: 8 chars, `[a-z0-9]`, non-sequential, globally unique, system-generated
- Public redirect endpoint: `GET /out/:public_id` — resolves OutLink, returns 302 redirect, enqueues click recording job
- `out-links.click` BullMQ job: records `link_click` AnalyticsEvent, emits `out-link.clicked` event
- Scheduled jobs: `out-links.purge` (archived OutLinks hard-deleted after 90 days)
- `AnalyticsService` implemented: `recordProfileView`, `recordLinkClick({click_id, out_link_id, account_id})` (idempotent on `click_id` — see `04-service-contracts.md`), `getProfileViewSummary`, `getLinkClickSummary`, `getAnalyticsSummary`
- `OutLinksService.resolveOutLink` generates `click_id` synchronously and enqueues the click job with `jobId = click_id`; the partial unique index on `OutLink` and the `AnalyticsEvent.id = click_id` primary key are both in place before this phase is considered complete
- `AnalyticsEventRepository` implemented (append-only)
- API endpoints implemented: `GET /analytics/summary`, `GET /analytics/profile`, `GET /analytics/out-links/:id`
- Scheduled job: `analytics.retention` (purge old AnalyticsEvent records)
- `account.deletion.cascade` BullMQ job fully implemented: (1) OutLinksService.getAccountOutLinkIds (read-only), (2) AnalyticsService.deleteAccountAnalytics, (3) OutLinksService.deleteAccountOutLinks, (4) ProfileService.deleteAccountData, (5) ConnectedAccountsService.deleteAccountConnectedAccounts, (6) AccountService.deleteQrCode (hard-deletes the `AccountQRCode` record and its SVG asset); all steps idempotent; see `10-background-jobs.md` — Job 7. The `AIService.deleteAccountAnalysisSessions` step is added in Phase 8 once `AnalysisSession` exists.
- Events emitted: `out-link.created`, `out-link.archived`, `out-link.clicked`

**OutLink creation model:** OutLinks are system-created, not user-facing. `ProfileService.addBlock` calls `OutLinksService.createOutLink` after block persistence: one OutLink for `button` blocks; one OutLink per entry in `content.accounts` for `social_icons` blocks (each with its own `connected_account_id`). Each social icon renders with its own `/out/{public_id}` tracked link.

**Cascade idempotency:** `account.deletion.cascade` Job 7 collects out_link_ids via `OutLinksService.getAccountOutLinkIds` (read-only) BEFORE any deletion. Analytics deletion runs before OutLink deletion. On retry, if OutLinks are already deleted, the read returns empty and analytics is a safe no-op. All cascade steps are idempotent; `AccountQRCode` record and SVG asset deletion is the final cascade step (after the Phase 8 `AnalysisSession` cleanup step is added).

**Complete when:** OutLink redirect flow works end-to-end; click recording is non-blocking; analytics events accumulate correctly; per-icon OutLink tracking works for social_icons blocks; account deletion cascade is fully idempotent.

---

## Phase 7 — Rendering and Caching

**Reference:** `04-service-contracts.md`, `05-api-contracts.md`, `09-caching-strategy.md`, `12-seo-and-integrations.md`

**Deliverables:**

- `RenderingService` implemented: assembles render object from Account + ProfileContent + Blocks + ConnectedAccounts + OutLinks + ImageAssets (all read-only); resolves reference blocks; constructs avatar URL via `StoragePlatform.buildPublicUrl`; resolves `button` and `social_icons` block links to `/out/{public_id}` tracked URLs via OutLinkRepository lookup by block_id; resolves `image` block URLs via ImageAssetRepository lookup by content.image_id then `StoragePlatform.buildPublicUrl(storage_key)`
- Public profile API endpoint: `GET /{username}` — profile view recording, render object assembly, HTML generation
- Two-tier cache implemented per `09-caching-strategy.md`:
  - Application Redis cache: `profile:public:{username}`, `profile:content:{account_id}`, TTL 60s
  - Edge CDN cache: HTTP response cache-control headers, max-age 60s
- Cache invalidation subscribers: `profile.*` and `block.*` events invalidate `profile:public:{username}` and `profile:content:{account_id}`
- SEO metadata generation integrated into rendering pipeline per `12-seo-and-integrations.md`
- GTM snippet injection integrated into HTML assembly per `12-seo-and-integrations.md`

**No publishing workflow:** Saved profile data is visible at public URL within ≤ 60 seconds (cache TTL). There is no draft/publish state.

**Complete when:** `GET /{username}` returns fully rendered profile with correct SEO metadata; cache invalidation triggers on profile updates; cache responds within TTL.

---

## Phase 8 — AI Platform Integration

**Reference:** `04-service-contracts.md`, `11-platform-services.md`, `08-security-model.md`

**Deliverables:**

- `AnalysisSessionRepository` implemented (entity 11 in `03-canonical-data-model.md`)
- `AIService` implemented with single `analyzeProfile(account_id)` method; builds the `AnalysisInputSnapshot`, builds an `AIProviderRequest`, and delegates to `Provider.execute(request)` for inference
- `AnalysisInputSnapshot` construction and normalization implemented per `11-platform-services.md` — deterministic key/array ordering, stable empty-value representation
- `analysis_version` and `output_schema_version` defined as Minime-owned semantic version string constants (`MAJOR.MINOR.PATCH`, e.g. `"1.0.0"`), co-located with `packages/config/ai-prompts.ts`; not environment-configurable, not provider-configurable
- `provider_key` and `model_key` stored on `AnalysisSession` purely as diagnostic metadata (debugging, support, monitoring, cost analysis, provider performance review, incident investigation); never read for reuse, validity, or business logic
- V1 AI scope: one capability — "Analyze My Profile" on-demand Analysis Session; no username suggestions, no social setup suggestions, no onboarding guidance
- Provider configured via `AI_PROVIDER`, `AI_MODEL_ID`, `AI_API_KEY` environment variables; implementation defaults in `.env.example`
- Prompt template for profile analysis implemented in `packages/config/ai-prompts.ts`
- Timeout enforced (`AI_TIMEOUT_MS`); timeout marks the Analysis Session as failed and returns empty suggestions; never throws to caller
- Input Hash computed from the normalized Analysis Input Snapshot; the latest completed Analysis Session is reused (Analysis Session Reuse) only when `input_hash`, `analysis_version`, and `output_schema_version` all match — no new `Provider.execute` call is made; reuse never depends on elapsed time
- AI output validated against the configured current `output_schema_version`; non-conforming output marks the Analysis Session as failed
- If `AI_API_KEY` is absent: `AIService.analyzeProfile` returns `{ status: 'disabled', suggestions: [] }` immediately; no Analysis Session is created. A provider failure/timeout/invalid-output instead returns `{ status: 'failed', analysis_session_id, suggestions: [] }` with a persisted failed session — these are the two distinct outcomes the API layer surfaces (see `05-api-contracts.md` — Part 9)
- `AIService.getLatestCompletedAnalysisSession` implemented; backs `GET /api/v1/account/ai/analysis/latest`
- API endpoints implemented: `POST /api/v1/account/ai/analysis`, `GET /api/v1/account/ai/analysis/latest` (see `05-api-contracts.md` — Part 9); rate limit (10/account/24h) applied to the `POST` endpoint
- AI surfaced in "Analyze My Profile" dashboard feature only; `AIService` is not invoked in registration, social setup, or onboarding flows
- AI failure handled gracefully: Analysis Session marked failed, returns empty suggestions, never throws to caller
- No automatic AI mutations: all suggestions require explicit user confirmation before domain service mutation executes
- `AIService` not invoked from background jobs, event handlers, or scheduled tasks
- `account.deletion.cascade` job (built in Phase 6) extended with `AIService.deleteAccountAnalysisSessions({ account_id })` — hard-deletes all `AnalysisSession` records for the account regardless of status; idempotent; see `10-background-jobs.md` — Job 7

**Complete when:** "Analyze My Profile" returns profile improvement suggestions; AI failure does not interrupt any product domain operation; timeout and Analysis Session Reuse (Input Hash, `analysis_version`, and `output_schema_version` match) are enforced; AI is absent from all non-explicit request flows.

---

## Phase 9 — Hardening

**Reference:** `07-validation-rules.md`, `08-security-model.md`

**Deliverables:**

- Rate limiting applied to all endpoints listed in `08-security-model.md` — Rate Limiting
- HTTP security headers configured: `X-Content-Type-Options`, `X-Frame-Options`, `Strict-Transport-Security`, referrer policy
- CORS allow-list configured (no wildcard origins in production)
- Input validation confirmed at all API boundaries (Zod schemas per `07-validation-rules.md`)
- Background job monitoring: failed jobs logged with alertable status
- Secrets confirmed absent from logs, responses, and job payloads
- Unit test coverage for all service methods
- Integration test coverage for all API endpoints
- End-to-end Playwright tests for critical user flows: registration, profile edit, public profile view, OutLink redirect

**Complete when:** All security requirements in `08-security-model.md` are satisfied; rate limiting verified under load; test suite passes.

---

## Phase 10 — Deployment

**Reference:** `01-technology-stack.md`, `02-repository-structure.md`

**Deliverables:**

- **Three Docker images**, not two: `apps/api` (NestJS, HTTP request handling only), `apps/web` (Next.js), and `apps/worker` (NestJS, same codebase/module graph as `apps/api`, running BullMQ processors only — no HTTP listener). All 15+ scheduled and event-triggered jobs in `10-background-jobs.md` (including the Deletion Outbox Dispatcher, Job 8) run exclusively in `apps/worker`, never inside an `apps/api` request-handling process. This is required because BullMQ job execution must not compete with or be coupled to HTTP request-handling capacity or lifecycle, and because `apps/api` may run multiple instances (see "Multi-Instance Policy" below) while job processing must not be duplicated per API instance.
- Nginx configured as reverse proxy, with a **complete** routing table (the previous two-route table omitted the public QR redirect, which the QR system depends on):
  - `/api/*` → NestJS (`apps/api`)
  - `/out/*` → NestJS (`apps/api`) — public OutLink redirect
  - `/qr/*` → NestJS (`apps/api`) — public QR redirect (`GET /qr/{qr_code_id}`, `05-api-contracts.md` — Part 10); omitting this route means the QR system's only public entry point resolves to Next.js and 404s
  - `/*` → Next.js (`apps/web`) — catch-all, including `/{username}`
- **Node.js version pinning:** the exact Node.js major.minor version (not "current LTS," which drifts) is pinned in an `.nvmrc` file at the repository root and in each Dockerfile's `FROM node:X.Y-slim` base image tag. `apps/api`, `apps/worker`, and `apps/web` all pin the identical version. Upgrading Node is a deliberate, reviewed change to these files, never an implicit consequence of "current LTS" moving.
- **Multi-instance policy:** `apps/api` may run more than one instance behind Nginx/a load balancer; `apps/web` may run more than one instance; `apps/worker` runs as one logical BullMQ consumer group — multiple `apps/worker` instances are permitted for throughput, since BullMQ's own per-job locking (not EventEmitter) is what prevents duplicate job execution across instances. No component relies on in-process EventEmitter state being visible to another instance — the one place V1 previously depended on that (account deletion cascade enqueue) is now guaranteed durably via `AccountDeletionOutbox` and the Deletion Outbox Dispatcher (`06-event-contracts.md` — "V1 Event Durability Classification"; `10-background-jobs.md` — Job 8), specifically because EventEmitter handlers registered in one `apps/api` instance never fire for events published by another instance.
- GitHub Actions deployment pipeline: build → test → push images → deploy
- Production environment variables configured (database, Redis, Object Storage, Google OAuth `client_id`/`client_secret`, Apple OAuth Services ID `client_id`/Team ID/Key ID/private key, JWT secret)
- **Database migrations:** applied in CI before deployment, using a strictly additive/backward-compatible migration style (add columns/tables nullable or with defaults; never drop or rename a column in the same deploy that stops writing to it) so that the currently-running previous-version instances keep working during a rolling deploy. **Rollback:** rolling back a deployment means redeploying the previous image tag against the same (forward-migrated) schema — migrations are never rolled back automatically; a migration that must be reverted is reverted by a new, forward-only corrective migration, consistent with the platform's general "never rewrite history" posture (`platform/events/event.lifecycle.specification.v1.md` — "Event Replacement" expresses the same principle for a different entity, and the same discipline applies here).
- Health check endpoints active: `GET /health` (API), `GET /health` (Worker — reports BullMQ queue connectivity, not HTTP readiness, since Worker has no HTTP listener), `GET /` (Web)
- Monitoring and alerting configured for: API error rate, background job failure rate, cache miss rate, database connection pool, `AccountDeletionOutbox` rows with `dispatched_at IS NULL` older than 1 hour (`10-background-jobs.md` — Job 8 — "Reconciliation")

**Complete when:** Production deployment completes; health checks pass (including Worker); `/qr/*` resolves correctly through Nginx; all Phase 1–9 functionality verified in production.

---

## Implementation Invariants

The following must hold across all phases. Violations must be corrected before the phase is considered complete.

| Invariant | Source |
|---|---|
| No `User`, `Profile`, `PublicProfile`, or `SocialAccount` entity | `03-canonical-data-model.md` |
| No `profile_id` foreign key anywhere | `03-canonical-data-model.md` |
| No Email OTP, no password authentication, no phone/SMS/WhatsApp OTP, no Facebook/X/LinkedIn login | `08-security-model.md` |
| Supported providers in V1: Google Sign-In and Apple Sign-In only; `provider` is always `google` \| `apple` | `authentication.policy.v1.md` |
| Authentication routes are provider-agnostic (`/auth/oauth/start`, `/auth/oauth/callback`, `/auth/oauth/register`, `/account/auth-providers*`); `provider` is request data, never a route segment; adding a provider adapter never changes route structure | `05-api-contracts.md` |
| Every account has ≥1 `AuthenticationIdentity`; the last remaining one is never removable, addressed by `auth_identity_id` | `04-service-contracts.md` |
| `email` is never the canonical authentication identity; identity is always `(provider, provider_subject)`; no automatic account merge exists | `authentication.policy.v1.md` |
| `AuthenticationIdentity` has no `display_name` column; provider-reported names live only in `provider_profile` | `03-canonical-data-model.md` |
| No publishing workflow; save = visible within cache TTL | Architecture |
| OutLinks are system-created, not user-API-created | `04-service-contracts.md` |
| Account deletion is immediate, final, irreversible | `03-canonical-data-model.md` |
| `state`/`nonce`/`oauth_handoff_token`/`link_handoff_token` never stored in PostgreSQL or Redis beyond their TTL; provider tokens never persisted | `08-security-model.md` |
| Refresh tokens never stored plain-text | `08-security-model.md` |
| Storage keys never exposed in API responses | `08-security-model.md` |
| AI mutations require explicit user confirmation | `11-platform-services.md` |
| Every controller → service → repository; no direct Prisma access | `03-canonical-data-model.md` |
| All validation via Zod schemas in `packages/validation` | `07-validation-rules.md` |
