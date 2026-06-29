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

- All 11 Prisma models defined exactly as specified in `03-canonical-data-model.md`:
  `Account`, `AuthCredential`, `OtpVerification`, `Session`, `UsernameReservation`, `ProfileContent`, `Block`, `ImageAsset`, `ConnectedAccount`, `OutLink`, `AnalyticsEvent`
- All enum types: `AccountStatus`, `AuthCredentialType`, `BlockType`, `SocialPlatform`, `AnalyticsEventType`, `DeviceCategory`, `OutLinkStatus`
- All indexes and uniqueness constraints as specified
- All repositories scaffolded (empty implementations, correct interfaces):
  `AccountRepository`, `AuthRepository`, `UsernameReservationRepository`, `ProfileContentRepository`, `BlockRepository`, `ImageAssetRepository`, `ConnectedAccountRepository`, `OutLinkRepository`, `AnalyticsEventRepository`
- Initial Prisma migration applied to development database

**Forbidden (per `03-canonical-data-model.md`):** No `User`, `Profile`, `PublicProfile`, or `SocialAccount` models.

**Complete when:** `prisma migrate dev` succeeds; all repositories injectable without errors.

---

## Phase 3 — Authentication System

**Reference:** `03-canonical-data-model.md`, `04-service-contracts.md`, `05-api-contracts.md`, `06-event-contracts.md`, `07-validation-rules.md`, `08-security-model.md`, `10-background-jobs.md`

**Deliverables:**

- `AuthService` implemented: `requestOtp`, `verifyOtp`, `googleSignIn`, `refreshSession`, `logout`, `logoutAll`
- Email OTP flow: OTP generation, synchronous email delivery via AuthEmailDelivery, hash storage, attempt tracking, OTP resend cooldown enforcement
- `AuthEmailDelivery` adapter implemented; V1 default uses Resend SDK; configured via `EMAIL_PROVIDER_API_KEY` and `EMAIL_FROM_ADDRESS` environment variables
- OTP delivery is synchronous: email sent before OtpVerification record is created; if delivery fails, no record is created and an error is returned
- Google Sign-In flow: ID token backend verification against Google public keys
- JWT access token issuance (short-lived, not stored)
- Refresh token rotation (hash stored in Session, plain text discarded after issuance)
- `AuthRepository` implemented: OtpVerification CRUD, Session CRUD, AuthCredential CRUD
- API endpoints implemented: `POST /auth/otp/request`, `POST /auth/otp/verify`, `POST /auth/google`, `POST /auth/session`, `POST /auth/logout`, `POST /auth/logout-all`
- Scheduled jobs: `auth.otp.cleanup`, `auth.session.cleanup`
- Auth guard (JWT validation middleware)
- Events emitted: `auth.otp.requested`, `auth.otp.verified`, `auth.session.refreshed`

**Security requirements (per `08-security-model.md`):**
- OTP codes never stored plain-text, never logged, never in responses, never in BullMQ payloads
- Refresh tokens never stored plain-text
- After 5 failed attempts, OtpVerification treated as invalid
- Google tokens never stored

**Complete when:** Full OTP and Google Sign-In flows succeed end-to-end; JWT guard rejects invalid tokens.

---

## Phase 4 — Account and Username

**Reference:** `03-canonical-data-model.md`, `04-service-contracts.md`, `05-api-contracts.md`, `06-event-contracts.md`, `07-validation-rules.md`, `08-security-model.md`

**Deliverables:**

- `AccountService` implemented: `createAccount`, `deleteAccount`, `getAccount`, `generateQrCode`, `deleteQrAsset`
- `AccountService.deleteAccount` implemented: sets Account.status = 'deleted', publishes `account.deleted` event; EventEmitter handlers perform synchronous session revocation and enqueue `account.deletion.cascade` BullMQ job
- `AccountService.generateQrCode` implemented: lazy QR generation via StoragePlatform.upload(assetType: 'qr'); storage key persisted in Account.qr_config.storage_key; called by GET /account/qr-code when storage_key is null
- `AccountService.updateQrCodeConfiguration` clears storage_key when config changes, forcing regeneration on next request
- `account.deletion.cascade` job scaffolded in this phase; fully implemented with all domain service calls in Phase 6 (after OutLinksService, AnalyticsService, ProfileService, and ConnectedAccountsService deletion commands are available)
- `UsernameService` implemented: `checkAvailability`, `reserveUsername`, `expireReservation`
- `UsernameReservationRepository` implemented
- Scheduled job: `username.reservation.expire`
- API endpoints implemented: `GET /usernames/availability`, `POST /usernames/reservations`, `GET /account`, `PATCH /account/settings`, `GET /account/qr-code`, `PATCH /account/qr-code`, `POST /account/deletion`
- Events emitted: `account.created`, `account.deleted`, `username.reserved`

**Invariants:**
- Username immutable after registration
- Account deletion is immediate, final, and irreversible
- Username permanently reserved after account deletion

**Complete when:** Registration flow completes (username → account created); account deletion cascade executes correctly and completely.

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
- `AnalyticsService` implemented: `recordProfileView`, `recordLinkClick`, `getProfileViewSummary`, `getLinkClickSummary`
- `AnalyticsEventRepository` implemented (append-only)
- API endpoints implemented: `GET /analytics/summary`, `GET /analytics/profile`, `GET /analytics/out-links/:id`
- Scheduled job: `analytics.retention` (purge old AnalyticsEvent records)
- `account.deletion.cascade` BullMQ job fully implemented: (1) OutLinksService.getAccountOutLinkIds (read-only), (2) AnalyticsService.deleteAccountAnalytics, (3) OutLinksService.deleteAccountOutLinks, (4) ProfileService.deleteAccountData, (5) ConnectedAccountsService.deleteAccountConnectedAccounts, (6) AccountService.deleteQrAsset; all steps idempotent; see `10-background-jobs.md` — Job 8
- Events emitted: `out-link.created`, `out-link.archived`, `out-link.clicked`

**OutLink creation model:** OutLinks are system-created, not user-facing. `ProfileService.addBlock` calls `OutLinksService.createOutLink` after block persistence: one OutLink for `button` blocks; one OutLink per entry in `content.accounts` for `social_icons` blocks (each with its own `connected_account_id`). Each social icon renders with its own `/out/{public_id}` tracked link.

**Cascade idempotency:** `account.deletion.cascade` Job 8 collects out_link_ids via `OutLinksService.getAccountOutLinkIds` (read-only) BEFORE any deletion. Analytics deletion runs before OutLink deletion. On retry, if OutLinks are already deleted, the read returns empty and analytics is a safe no-op. All cascade steps are idempotent; QR storage asset deletion is the final cascade step.

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

- `AiPlatform` service implemented: `suggestUsernames`, `suggestSocialSetup`, `suggestProfileImprovements`, `generateOnboardingGuidance`
- Provider configured via `AI_PROVIDER`, `AI_MODEL_ID`, `AI_API_KEY` environment variables; implementation defaults in `.env.example`
- Prompt templates implemented in `packages/config/ai-prompts.ts`
- Rate limiting enforced per account (`AI_MAX_REQUESTS_PER_MINUTE`); timeout enforced (`AI_TIMEOUT_MS`)
- If `AI_API_KEY` is absent: all methods return empty arrays silently
- AI suggestions surfaced in the relevant API flows (username selection during registration, social setup, profile onboarding)
- AI failure handled gracefully: returns empty array, never throws to caller
- No automatic AI mutations: all AI suggestions require explicit user confirmation before domain service mutation executes

**Complete when:** AI suggestions surface in registration and onboarding flows; AI failure does not interrupt any product domain operation; rate limiting and timeout are enforced.

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

- Docker images built for `apps/api` and `apps/web`
- Nginx configured as reverse proxy: routes `/api/*` to NestJS, `/*` to Next.js, `/out/*` to NestJS
- GitHub Actions deployment pipeline: build → test → push images → deploy
- Production environment variables configured (database, Redis, Object Storage, Google OAuth, JWT secret, GTM container ID)
- Database migrations applied in CI before deployment
- Health check endpoints active: `GET /health` (API), `GET /` (Web)
- Monitoring and alerting configured for: API error rate, background job failure rate, cache miss rate, database connection pool

**Complete when:** Production deployment completes; health checks pass; all Phase 1–9 functionality verified in production.

---

## Implementation Invariants

The following must hold across all phases. Violations must be corrected before the phase is considered complete.

| Invariant | Source |
|---|---|
| No `User`, `Profile`, `PublicProfile`, or `SocialAccount` entity | `03-canonical-data-model.md` |
| No `profile_id` foreign key anywhere | `03-canonical-data-model.md` |
| No password authentication, no phone OTP, no SMS | `08-security-model.md` |
| No publishing workflow; save = visible within cache TTL | Architecture |
| OutLinks are system-created, not user-API-created | `04-service-contracts.md` |
| Account deletion is immediate, final, irreversible | `03-canonical-data-model.md` |
| OTP codes never stored plain-text | `08-security-model.md` |
| Refresh tokens never stored plain-text | `08-security-model.md` |
| Storage keys never exposed in API responses | `08-security-model.md` |
| AI mutations require explicit user confirmation | `11-platform-services.md` |
| Every controller → service → repository; no direct Prisma access | `03-canonical-data-model.md` |
| All validation via Zod schemas in `packages/validation` | `07-validation-rules.md` |
