# Minime V1 — Architecture PR Approval Decisions

## Status

Approved. Canonical registry.

---

## Purpose

This document is the canonical registry of every implementation-enabling engineering decision approved after the Minime V1 architecture freeze (`Architecture-Product-Definition/docs/MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`).

Every decision recorded here:

- implements or clarifies an already-frozen product/architecture rule for the purpose of engineering execution,
- introduces no new Product Domain, no new product behavior, and no product scope beyond `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`,
- is traceable to the specific implementation and/or architecture files that define it and depend on it.

This document does not redesign V1. It does not introduce V2 functionality. It is not itself a source of product behavior — the files it references remain the authoritative specifications. This document exists to make the boundary between "frozen architecture" and "implementation-enabling clarification" explicit and auditable.

Any future addition to this repository that introduces a new entity, background job, deployable, repository contract, transaction coordinator, advisory lock, unique index, or security boundary must be recorded here before or alongside its introduction, per the governance rule in `implementation/README.md` — "Post-Freeze Clarification Policy."

---

## Registry Format

**Governing note (Phase A.5 — G-06):** The template below was expanded during the Repository Governance Hardening phase so that a future reader can understand what changed, what it replaced, and what it affects without cross-referencing any other document. Every entry from APD-001 onward — including the ten entries approved before this expansion — has been retrofitted to the expanded template.

Each decision below is recorded with:

- **Decision ID** — stable identifier for cross-referencing
- **Decision Name**
- **Status** — Approved (all entries in this document are approved; no other status is recorded here)
- **Classification** — the category of implementation-enabling clarification (Entity / Background Job / Deployable / Repository Contract / Transaction Coordinator / Advisory Lock / Unique Index / Security Boundary / API Contract / Architecture Correction)
- **Why it was required**
- **Why it is NOT an architectural redesign** — this field satisfies the "Reason this is not a redesign" requirement; no separate duplicate field is used
- **Supersedes** — the specific prior text, behavior, or (for APD-001 through APD-010) "None" if the decision is a net-new addition rather than a correction of prior text
- **Scope**
- **Explicit Non-Goals**
- **Affected Canonical Documents** — the `Architecture-Product-Definition/` (or root-level Map/registry) documents this decision is grounded in or corrects. An entry with no direct Architecture-tier document is explicitly recorded as such, never left blank.
- **Affected Implementation Documents** — the `implementation/` documents that define or operationalize this decision
- **Dependent Documents** — every other document (Architecture or Implementation) whose correctness assumes this decision holds
- **Migration Impact** — what changes for anyone building against the prior state
- **Backward Compatibility** — whether the change is compatible with data/behavior that existed before the decision

---

## APD-001 — AccountDeletionOutbox

**Status:** Approved

**Classification:** Entity / Durability Mechanism

**Why it was required:** Account deletion must trigger cross-domain cascade cleanup (sessions, analytics, out links, profile data, connected accounts, analysis sessions, QR code) even if the process crashes between committing `Account.status = 'deleted'` and the best-effort `account.deleted` EventEmitter notification firing. NestJS EventEmitter is in-process, non-durable, and provides no delivery guarantee across a crash or restart (`06-event-contracts.md` — "Technology: NestJS EventEmitter"). Without a durable mechanism, a crash at the wrong instant would leave a deleted account's data uncleaned indefinitely.

**Why it is NOT an architectural redesign:** `Architecture-Product-Definition/platform/events/events.architecture.specification.v1.md` ("Failure Philosophy") and `event.lifecycle.specification.v1.md` ("Product correctness always has higher priority than Event recording") already establish, at the architecture level, that business correctness must never depend solely on Event delivery. `AccountDeletionOutbox` is the narrow, single-purpose implementation of that pre-existing architectural rule, scoped only to the `deleteAccount` command. It does not change account deletion semantics (still immediate, final, and irreversible), does not introduce a new Product Domain, is not part of the Events Platform, and does not generalize into an event store.

**Supersedes:** None — net-new durability mechanism; no prior document described an account-deletion durability guarantee.

**Scope:** One row per deleted account (`account_id` PK), inserted in the same transaction as `Account.status = 'deleted'`. Read by the Deletion Outbox Dispatcher background job (Job 8, every 30 seconds), which enqueues `account.deletion.cascade` for any undispatched row.

**Explicit Non-Goals:** Not an event store. Not a general saga/orchestration framework. Not used by any command other than `deleteAccount`. Not a user-facing feature. Does not replace or duplicate the `AuditLog` entity.

**Affected Canonical Documents:**
- `Architecture-Product-Definition/platform/events/events.architecture.specification.v1.md` — "Failure Philosophy" (grounding principle; not modified by this decision)
- `Architecture-Product-Definition/platform/events/event.lifecycle.specification.v1.md` — "Product correctness always has higher priority than Event recording" (grounding principle; not modified)
- `Architecture-Product-Definition/account/account.deletion.policy.v1.md` (cascade this mechanism durably triggers; not modified by this decision, but see APD-015 for a later correction to this same document)

**Affected Implementation Documents:**
- `implementation/03-canonical-data-model.md` — entity 13
- `implementation/04-service-contracts.md` — `AccountService.deleteAccount`
- `implementation/10-background-jobs.md` — Job 8 (Deletion Outbox Dispatcher), Job 7 (Account Deletion Cascade)
- `implementation/06-event-contracts.md` — "V1 Event Durability Classification"

**Dependent Documents:**
- `implementation/08-security-model.md` — "Session Security" (session revocation guarantee)
- `implementation/13-implementation-plan.md` — Phase 4, Phase 10 (monitoring: outbox rows with `dispatched_at IS NULL` older than 1 hour)
- `implementation/02-repository-structure.md` — `apps/worker` (Job 8 execution)

**Migration Impact:** None — this is the first specification of the mechanism; there is no prior behavior to migrate from.

**Backward Compatibility:** Fully compatible. Introducing this table and job does not change the observable outcome of account deletion (still immediate, final, irreversible) — it only guarantees that outcome completes even after a process crash.

---

## APD-002 — ImageAsset

**Status:** Approved

**Classification:** Entity

**Why it was required:** Image Blocks need a way to reference an uploaded binary image (Object Storage key, MIME type, dimensions, file size) distinct from the single `ProfileContent.avatar_storage_key`. Without a dedicated entity, image block content would need to embed storage keys directly, contradicting the storage-key-opacity rules already in `08-security-model.md`.

**Why it is NOT an architectural redesign:** `ImageAsset` is a metadata-only record mirroring the pattern the frozen architecture already uses for avatar storage (`ProfileContent.avatar_storage_key`) and is already referenced by `Architecture-Product-Definition/rendering/block.renderers.specification.v1.md` ("Image Renderer") and `Architecture-Product-Definition/platform/storage/asset.processing.specification.v1.md`. It explicitly is not a media library — no search, browse, gallery, album, or folder capability exists or is implied.

**Supersedes:** None — net-new entity; no prior document modeled image block storage.

**Scope:** `id`, `account_id`, `storage_key`, `mime_type`, `width`, `height`, `file_size`, `created_at`. Hard Delete only (no `deleted_at`, no `updated_at`). Owned by the Profile domain.

**Explicit Non-Goals:** No media library, gallery, album, or folder. No alt-text storage (alt is always `""` per the frozen rendering spec). No version history. No cross-account sharing.

**Affected Canonical Documents:**
- `Architecture-Product-Definition/rendering/block.renderers.specification.v1.md` — Image Renderer (already assumed this entity's existence conceptually; not modified by this decision)
- `Architecture-Product-Definition/platform/storage/asset.processing.specification.v1.md` (grounding principle for storage-key opacity; not modified)
- `Architecture-Product-Definition/blocks/image.block.specification.v1.md` — Content Shape (`image_id` reference; already consistent, not modified)

**Affected Implementation Documents:**
- `implementation/03-canonical-data-model.md` — entity 7
- `implementation/04-service-contracts.md` — `ProfileService.uploadImage`, `deleteImage`
- `implementation/05-api-contracts.md` — Part 5 (`POST /profile/images`, `DELETE /profile/images/{imageId}`)
- `implementation/07-validation-rules.md` — "ImageAsset Validation"

**Dependent Documents:**
- `Architecture-Product-Definition/rendering/block.renderers.specification.v1.md` — Image Renderer
- `implementation/11-platform-services.md` — Storage Platform
- `implementation/10-background-jobs.md` — Job 7 (cascade deletion step 5)

**Migration Impact:** None — first specification of the entity.

**Backward Compatibility:** Fully compatible. No prior entity or field is renamed, removed, or reinterpreted.

---

## APD-003 — AuditLog

**Status:** Approved

**Classification:** Entity

**Why it was required:** `Architecture-Product-Definition/account-management/audit.logging.policy.v1.md` already mandates operational audit logging for account, authentication, connected-account, profile, and administrative changes. V1 implementation needed a concrete persisted entity, a closed `action` catalog, and a single write-path service to implement that pre-approved policy.

**Why it is NOT an architectural redesign:** This directly implements the existing, approved `audit.logging.policy.v1.md`. The V1 logged-event catalog is a narrowing, not an expansion, of that policy: it excludes `account_updated` (settings are not itemized), `recovery_email_changed` (Recovery Email does not exist in V1), `page_*` (there is no Page entity distinct from the single public profile), and individual `block_*` events (already covered by `Block` row history) — each exclusion documented against an already-true V1 scope fact, not a new restriction invented at implementation time.

**Supersedes:** Narrows (does not contradict) the event catalog originally listed in `audit.logging.policy.v1.md` — see the "Governing note" added to that document's "Logged Events" section during Phase A.5 (G-04), which now states the same narrowing directly at its source.

**Scope:** `log_id`, `account_id`, `actor_account_id`, `action`, `entity_type`, `entity_id`, `old_value`, `new_value`, `metadata`, `created_at`. Written synchronously (never via EventEmitter) by `AccountService`, `AuthService`, `ConnectedAccountsService`, `ProfileService`. 12-month retention, then hard delete.

**Explicit Non-Goals:** No admin UI in V1. No user-facing or account-facing access. No rollback, restore, undo, or revision recovery capability. Not a versioning system. Not a substitute for `AccountDeletionOutbox` or any other durability mechanism.

**Affected Canonical Documents:**
- `Architecture-Product-Definition/account-management/audit.logging.policy.v1.md` — source policy; "Logged Events" section corrected directly during Phase A.5 to match this entity's actual catalog

**Affected Implementation Documents:**
- `implementation/03-canonical-data-model.md` — entity 14
- `implementation/04-service-contracts.md` — `AuditLogService`
- `implementation/10-background-jobs.md` — Job 9 (Audit Log Retention)

**Dependent Documents:**
- `implementation/13-implementation-plan.md` — Phase 4

**Migration Impact:** None — first specification of the entity; the Phase A.5 correction to `audit.logging.policy.v1.md` only removed references to events that were never implemented.

**Backward Compatibility:** Fully compatible.

---

## APD-004 — apps/worker

**Status:** Approved

**Classification:** Deployable

**Why it was required:** V1 defines 15+ BullMQ background jobs, including the Deletion Outbox Dispatcher (Job 8), which must run on a fixed schedule independent of HTTP request volume. Running job processors inside the same process as `apps/api` would couple job execution capacity to HTTP request capacity, and would risk a job being processed once per `apps/api` instance if `apps/api` is horizontally scaled.

**Why it is NOT an architectural redesign:** `apps/worker` introduces no new Product Domain and no new business logic. It shares the same Product Domain service layer, repositories, and Prisma schema as `apps/api` (`02-repository-structure.md` — §3.3), and job workers call Product Domain services exactly as `apps/api` controllers do. It is a deployment/infrastructure decision — a separate deployable process/container — not a product or data-ownership change. The monorepo's `apps/` convention already anticipated multiple apps.

**Supersedes:** None — net-new deployable; no prior document specified a two-app (`apps/web`, `apps/api`) or single-app deployment topology that this contradicts.

**Scope:** `apps/worker` executes BullMQ job processors and scheduled (cron-style) job triggers only. No HTTP listener beyond its own health check. Runs as one logical BullMQ consumer group; multiple instances are permitted for throughput because BullMQ's own per-job locking (not EventEmitter) prevents duplicate execution across instances.

**Explicit Non-Goals:** Not a new Product Domain. Not a microservice with independent data ownership. Not a place for new business logic distinct from what `apps/api` already implements. Not a public or internal HTTP surface.

**Affected Canonical Documents:** None directly — this is a deployment-topology decision. Its architectural grounding is the general Platform Architecture principle that Runtime Execution (deferred/scheduled behaviors) belongs to the deployment environment (`Architecture-Product-Definition/platform/platform.architecture.specification.v1.md` — "Runtime Execution"), which is not modified by this decision.

**Affected Implementation Documents:**
- `implementation/02-repository-structure.md` — §3.3
- `implementation/13-implementation-plan.md` — Phase 10 ("Three Docker images," "Multi-Instance Policy")

**Dependent Documents:**
- `implementation/10-background-jobs.md` — "Job Flow" ("Job workers must go through Product Domain services, not directly to repositories or Prisma")
- `implementation/01-technology-stack.md` — Node.js version pinning across `apps/api`, `apps/worker`, `apps/web`

**Migration Impact:** None — first specification of this deployable.

**Backward Compatibility:** Fully compatible; no existing API, entity, or business rule changes.

---

## APD-005 — Authentication Provider Adapter

**Status:** Approved

**Classification:** Security Boundary / Repository Contract (Extensibility Pattern)

**Why it was required:** V1 needed a concrete mechanism so Google Sign-In and Apple Sign-In could each be implemented without provider-specific branching inside `AuthService`, and so that a future provider could be added without redesigning routes, the `AuthenticationIdentity` schema, or `AuthService`'s public command signatures.

**Why it is NOT an architectural redesign:** This is a direct implementation of a dedicated, pre-existing architecture document — `Architecture-Product-Definition/account/authentication.provider.system.specification.v1.md` — whose stated design principles ("Provider Independence," "Stable Core," "Provider Isolation," "Identity First," "Single Source of Truth") already require exactly this adapter boundary. The implementation layer (`04-service-contracts.md`) implements the adapter contract that document already mandates; it does not invent a new architectural concept.

**Supersedes:** None — this is a direct implementation of an already-approved architecture document, not a correction of any prior text.

**Scope:** `AuthenticationProviderAdapter` type contract (`buildAuthorizationUrl`, `exchangeCodeForTokens`, `validateIdToken`, `normalizeIdentity`); `resolveProviderAdapter(provider)`; `GoogleAuthenticationProviderAdapter` and `AppleAuthenticationProviderAdapter` as the two V1 implementations.

**Explicit Non-Goals:** Does not add a third provider in V1. Does not change the `AuthenticationIdentity` schema, API route structure, or `Session` model. Does not permit any provider value outside the V1 allowlist (`google`, `apple`).

**Affected Canonical Documents:**
- `Architecture-Product-Definition/account/authentication.provider.system.specification.v1.md` — source architecture (not modified; already required this exact adapter boundary)
- `Architecture-Product-Definition/account/authentication.policy.v1.md` — "Provider Adapter Architecture" (consistent with, not modified)

**Affected Implementation Documents:**
- `implementation/04-service-contracts.md` — AuthService "Provider Adapter Contract"
- `implementation/08-security-model.md` — "Provider Adapter Secret Ownership"

**Dependent Documents:**
- `implementation/05-api-contracts.md` — Part 2 (Authentication Domain API)
- `implementation/07-validation-rules.md` — "Authentication"
- `implementation/13-implementation-plan.md` — Phase 3

**Migration Impact:** None — first concrete realization of an already-approved architecture pattern.

**Backward Compatibility:** Fully compatible; adding a future third adapter requires no change to this contract, per its own stated extensibility goal.

---

## APD-006 — AnalysisSession API

**Status:** Approved

**Classification:** Entity / API Contract

**Why it was required:** `Architecture-Product-Definition/platform/ai/ai.architecture.specification.v1.md` already defines the Analysis Session concept and its reuse semantics at the architecture level. Implementation needed the concrete Prisma entity (field types, indexes, constraints) and the exact synchronous REST surface (`POST /api/v1/account/ai/analysis`, `GET /api/v1/account/ai/analysis/latest`) to expose the already-approved "Analyze My Profile" capability.

**Why it is NOT an architectural redesign:** The entity concept, its reuse rule (Input Hash + `analysis_version` + `output_schema_version` match), and its status lifecycle are already specified in `ai.architecture.specification.v1.md` — "Analysis Session." The implementation layer only adds field-level schema, database indexes, and the two REST endpoints already implied by the architecture's "on-demand, synchronous" framing. No new AI capability is introduced.

**Supersedes:** None — direct realization of an already-approved architecture concept, not a correction.

**Scope:** `AnalysisSession` Prisma entity (`03-canonical-data-model.md` — entity 11); `POST /api/v1/account/ai/analysis` (synchronous, blocks until complete/failed/reused); `GET /api/v1/account/ai/analysis/latest` (read-only); rate limit of 10 requests per account per rolling 24-hour window on the `POST` endpoint.

**Explicit Non-Goals:** No polling endpoint. No `applySuggestion`/`acceptSuggestion`/`rejectSuggestion` endpoint (V2 scope). No multi-provider routing. No background or scheduled AI execution. No AI chat.

**Affected Canonical Documents:**
- `Architecture-Product-Definition/platform/ai/ai.architecture.specification.v1.md` — source architecture (not modified; already defined the Analysis Session concept and reuse semantics)

**Affected Implementation Documents:**
- `implementation/03-canonical-data-model.md` — entity 11
- `implementation/05-api-contracts.md` — Part 9
- `implementation/11-platform-services.md` — "AI Platform"

**Dependent Documents:**
- `implementation/08-security-model.md` — "AI Security"
- `implementation/10-background-jobs.md` — Job 7, step 7 (`AIService.deleteAccountAnalysisSessions`)

**Migration Impact:** None — first concrete Prisma-level and API-level realization of the already-approved concept.

**Backward Compatibility:** Fully compatible.

---

## APD-007 — GTM Sandbox Isolation

**Status:** Approved

**Classification:** Security Boundary

**Why it was required:** Integrations architecture already approves Google Tag Manager as V1's sole integration provider, but a concrete rendering-layer mechanism was needed to prevent a user-configured GTM container (which can contain arbitrary third-party JavaScript via Custom HTML tags) from executing against Minime's own origin, cookies, or session.

**Why it is NOT an architectural redesign:** The sandboxed-iframe containment boundary is already mandated, in full, by `Architecture-Product-Definition/integrations/providers/google-tag-manager.specification.v1.md` — "Security Principles" ("Containment boundary": `sandbox="allow-scripts"` only, no `allow-same-origin`). `implementation/12-seo-and-integrations.md` only pins the exact HTML snippet (iframe placement, `srcdoc` encoding, omission of the `<noscript>` fallback) that satisfies the already-approved rule. No new integration provider or capability is introduced.

**Supersedes:** None — direct realization of an already-mandated security boundary, not a correction.

**Scope:** GTM snippet injected as a sandboxed `<iframe>` immediately after the opening `<body>` tag, `sandbox="allow-scripts"` only (never `allow-same-origin`, `allow-top-navigation`, `allow-popups`, or `allow-forms`); conditional on `Account.settings.gtm_container_id` being present and non-empty; injected server-side as part of the cached render response.

**Explicit Non-Goals:** Not a general integrations framework. No additional providers (Meta Pixel, LinkedIn Insight Tag, custom webhooks) in V1. No `<noscript>` fallback in V1.

**Affected Canonical Documents:**
- `Architecture-Product-Definition/integrations/providers/google-tag-manager.specification.v1.md` — "Security Principles" (not modified; already mandated the sandboxed-iframe containment boundary in full)

**Affected Implementation Documents:**
- `implementation/12-seo-and-integrations.md` — "Google Tag Manager (GTM)," "GTM Security Rules"

**Dependent Documents:**
- `implementation/09-caching-strategy.md` — cache invalidation on `gtm_container_id` change
- `implementation/03-canonical-data-model.md` — `Account.settings` field structure
- `implementation/07-validation-rules.md` — "Account Settings"

**Migration Impact:** None — first concrete HTML-level realization of an already-mandated security rule.

**Backward Compatibility:** Fully compatible.

---

## APD-008 — PostgreSQL Advisory Locks

**Status:** Approved

**Classification:** Transaction Coordinator / Concurrency Mechanism

**Why it was required:** Several invariants the frozen architecture already requires (max one identity block per account, unique `sort_order` per account, `MAX_BLOCKS_PER_ACCOUNT`, exactly one `Account` created per reserved username) involve a read-then-insert or read-then-count-then-insert sequence that plain `UNIQUE` constraints on two independent tables cannot make race-free. Without a shared lock, concurrent requests (e.g., the same user in two browser tabs, or two registration attempts racing the same username reservation window) could violate these invariants.

**Why it is NOT an architectural redesign:** `pg_advisory_xact_lock` is a standard PostgreSQL concurrency-control primitive applied per `account_id` (Block/ConnectedAccount mutations) or per normalized-username hash (registration). It changes no product-visible behavior, no business rule, and no entity shape — it only guarantees, under concurrency, invariants the architecture already states must hold. It does not weaken the platform's Last-Write-Wins concurrency model for ordinary field values (`data.architecture.specification.v1.md` — "Concurrency"); it protects only count/uniqueness invariants.

**Supersedes:** None — a pure concurrency-control mechanism added without changing any documented invariant.

**Scope:** `pg_advisory_xact_lock(hashtext(account_id))` held for the duration of `addBlock`/`updateBlock`/`deleteBlock`/`reorderBlocks` and the identically-keyed `ConnectedAccount` mutations (same lock, serializing Block and ConnectedAccount mutations against each other too). `pg_advisory_xact_lock(hashtext(normalized_username))` held by `UsernameService.reserveUsername` and `AuthService.completeRegistrationWithOAuth`.

**Explicit Non-Goals:** Not a general distributed-locking framework. Not used outside the two documented cases. Does not replace the partial unique index backstops (APD-009) — both mechanisms are defense-in-depth for the same invariants.

**Affected Canonical Documents:**
- `Architecture-Product-Definition/platform/data/data.architecture.specification.v1.md` — "Concurrency" (Last-Write-Wins model; not modified — this decision protects count/uniqueness invariants only, not ordinary field-value concurrency)
- `Architecture-Product-Definition/blocks/block.system.specification.v1.md` — identity block instance limit (invariant enforced, not changed)

**Affected Implementation Documents:**
- `implementation/03-canonical-data-model.md` — Block "Concurrency," ConnectedAccount "Concurrency"
- `implementation/04-service-contracts.md` — `UsernameService.reserveUsername`, AuthService "Registration Transaction Coordination"

**Dependent Documents:**
- `implementation/13-implementation-plan.md` — Phase 4, Phase 5

**Migration Impact:** None — first specification of this concurrency mechanism.

**Backward Compatibility:** Fully compatible; no product-visible behavior changes.

---

## APD-009 — Partial Unique Indexes

**Status:** Approved

**Classification:** Unique Index

**Why it was required:** Several invariants are conditional on soft-delete or status state — e.g., "only one active identity block of a given type," "no two active blocks share a `sort_order`," "at most one active OutLink per clickable identity." A plain `UNIQUE` constraint cannot express uniqueness scoped to "non-deleted" or "active" rows only; a global unique constraint would incorrectly block re-creating a block of a type whose prior instance was already soft-deleted.

**Why it is NOT an architectural redesign:** These indexes enforce invariants the architecture already states — "max 1 identity block per account" (`03-canonical-data-model.md`, itself derived from `blocks/block.system.specification.v1.md`) and "at most one active OutLink per clickable identity" (derived from `out-links/out-link.model.specification.v1.md`) — as a database-level, defense-in-depth backstop to the application-level transaction coordination in `04-service-contracts.md`. No invariant's meaning changes; only its enforcement layer is added.

**Supersedes:** None — an enforcement-layer addition for invariants already stated; no prior text is corrected.

**Scope:**
- `Block`: `UNIQUE (account_id, type) WHERE type IN ('avatar', 'name', 'bio') AND deleted_at IS NULL`
- `Block`: `UNIQUE (account_id, sort_order) WHERE deleted_at IS NULL`
- `ConnectedAccount`: `UNIQUE (account_id, sort_order)` (no partial clause needed — Hard Delete means no `deleted_at` to condition on)
- `OutLink`: `UNIQUE (block_id, connected_account_id) WHERE status = 'active'`

**Explicit Non-Goals:** Not a substitute for the application-level transaction coordination in `04-service-contracts.md` ("Block + OutLink Transaction Coordination"). Does not change any entity's lifecycle or status model.

**Affected Canonical Documents:**
- `Architecture-Product-Definition/blocks/block.system.specification.v1.md` — "max 1 identity block per account" (invariant enforced, not changed)
- `Architecture-Product-Definition/out-links/out-link.model.specification.v1.md` — "at most one active OutLink per clickable identity" (invariant enforced, not changed)

**Affected Implementation Documents:**
- `implementation/03-canonical-data-model.md` — Block, ConnectedAccount, OutLink "Constraints"
- `implementation/13-implementation-plan.md` — Phase 2

**Dependent Documents:**
- `implementation/04-service-contracts.md` — `ProfileService.addBlock`/`updateBlock`, "Block + OutLink Transaction Coordination"

**Migration Impact:** None — first specification of these indexes.

**Backward Compatibility:** Fully compatible; database-level backstop only, no business rule changes.

---

## APD-010 — OutLink Rendering Through /out/{public_id}

**Status:** Approved

**Classification:** Repository Contract / Rendering Rule

**Why it was required:** The architecture already defines `/out/{public_id}` as the canonical outbound routing surface. Implementation needed to pin down the exact lookup key `RenderingService` uses to resolve the active OutLink for a given block/icon (`(block_id, connected_account_id)`), and the precise omission rule when no active OutLink exists, so that raw destination URLs never leak into public render payloads.

**Why it is NOT an architectural redesign:** `/out/{public_id}` is already the canonical architecture-level routing pattern, specified in `Architecture-Product-Definition/out-links/out-link.routing.specification.v1.md` and `out-link.system.specification.v1.md`. The implementation decision only fixes the concrete repository lookup and the link-omission behavior needed to satisfy the pre-existing rule that "raw destination URLs must not appear in public render payloads."

**Supersedes:** None — pins down an implementation detail of an already-canonical routing pattern; no prior text is corrected.

**Scope:** For `button` blocks, `RenderingService` looks up one active OutLink by `(block_id, connected_account_id = null)`; for `social_icons` blocks, one active OutLink per icon by `(block_id, connected_account_id)`. Each renders as `/out/{public_id}`. If no active OutLink exists for a button or a specific icon, that link is omitted entirely from the rendered output — it must never fall back to the raw URL.

**Explicit Non-Goals:** Does not introduce new redirect behavior, new analytics event types, or change the existing 90-day archive-then-purge lifecycle already defined for `OutLink`.

**Affected Canonical Documents:**
- `Architecture-Product-Definition/out-links/out-link.routing.specification.v1.md` — source architecture (not modified; already defined `/out/{public_id}` as canonical)
- `Architecture-Product-Definition/out-links/out-link.system.specification.v1.md` (consistent with, not modified)

**Affected Implementation Documents:**
- `implementation/04-service-contracts.md` — `RenderingService`, `OutLinksService`
- `implementation/05-api-contracts.md` — Part 7, Part 10

**Dependent Documents:**
- `implementation/03-canonical-data-model.md` — `OutLink` entity, indexes
- `implementation/09-caching-strategy.md` — profile render cache

**Migration Impact:** None — first concrete specification of the lookup key.

**Backward Compatibility:** Fully compatible.

---

## APD-011 — Theme Customization Deferred to V2

**Status:** Approved

**Classification:** Architecture Correction (Scope Narrowing)

**Why it was required:** Four approved architecture documents (`appearance/themes/theme.customization.specification.v1.md`, `theme.definition.specification.v1.md`, `theme.selection.specification.v1.md`, `appearance/design.editor.specification.v1.md`) described a complete theme-wide visual customization engine (background, colors, typography, spacing, radius, borders, shadows, animations) as current V1 capability, with concrete field names (e.g. `primary_color`, `radius_xs`..`radius_xl`, `shadow_style`, `animation_preset`). The approved implementation (`implementation/03-canonical-data-model.md`) states `appearance_config.customizations` is always `{}` in V1 and defines no validation schema for any of these fields. This is a direct contradiction between frozen architecture and approved implementation, identified during the Phase A architecture-synchronization audit.

**Resolution:** V1 supports **Theme selection only** (choosing one Theme from the Theme Catalog). Theme-wide customization of any kind is confirmed as **V2 Scope**. This is a scope narrowing, not a scope expansion — it makes the documents match behavior that was already true in the shipped data model; it does not remove any capability that was actually available to users.

**Why it is NOT an architectural redesign:** No V1 capability is removed by this decision, because the customization capability was never implemented in the first place — only the documentation is corrected to match reality. Theme Selection, Theme Constraints (as applied to V1 block-level style overrides), and the Theme Catalog itself are unaffected and remain exactly as previously specified.

**Supersedes:** The prior, undifferentiated text in the six Appearance documents listed under "Scope" below that described theme-wide customization (colors, typography, spacing, radius, borders, shadows, animations, background) as present-tense V1 capability. That text is preserved in each document as the V2 target design, now explicitly labeled, rather than deleted.

**Scope:** All four documents listed above have been updated with explicit "V2 Scope" notices and per-section tags. `appearance/appearance.system.specification.v1.md` and `appearance/appearance.state.specification.v1.md` were also updated to add scope notices, since both described "Theme Customization" as a current Appearance responsibility. `settings/settings.categories.specification.v1.md` ("Visual customization" under the Appearance category) was also tagged during the Phase A.5 validation pass. Block-level style overrides (`block-styling/*`), which are a separate, narrower, already-approved V1 mechanism, are explicitly distinguished from theme-wide customization in all updated documents and are unaffected by this decision.

**Explicit Non-Goals:** Does not remove Theme Selection. Does not remove block-level style overrides. Does not define the eventual V2 customization API, schema, or validation rules — those remain to be specified when V2 is scheduled. Does not retroactively change `implementation/03-canonical-data-model.md`, which was already correct.

**Affected Canonical Documents:**
- `Architecture-Product-Definition/appearance/themes/theme.customization.specification.v1.md` — rewritten (canonical Owner of the V1-vs-V2 customization scope statement)
- `Architecture-Product-Definition/appearance/themes/theme.definition.specification.v1.md` — updated
- `Architecture-Product-Definition/appearance/themes/theme.selection.specification.v1.md` — updated
- `Architecture-Product-Definition/appearance/design.editor.specification.v1.md` — updated
- `Architecture-Product-Definition/appearance/appearance.system.specification.v1.md` — updated
- `Architecture-Product-Definition/appearance/appearance.state.specification.v1.md` — updated
- `Architecture-Product-Definition/settings/settings.categories.specification.v1.md` — updated

**Affected Implementation Documents:** None modified — Implementation was already correct.
- `implementation/03-canonical-data-model.md` — `ProfileContent.appearance_config.customizations` (reference only; confirms this decision, not modified)

**Dependent Documents:**
- `implementation/04-service-contracts.md` — `ProfileService.updateAppearance`
- `implementation/07-validation-rules.md` — Profile Content / Appearance validation

**Migration Impact:** Documentation-only. No engineer building against `implementation/` needs to change anything; engineers who had started building UI against the Appearance architecture documents' literal customization description must stop and treat those sections as V2 Scope only.

**Backward Compatibility:** Fully compatible — no implementation behavior changes; only the architecture documents' stated scope is corrected to match what was already true.

---

## APD-012 — Session Model: JWT Access Token + Rotating Refresh Token, With Session Management

**Status:** Approved

**Classification:** Architecture Correction (Implementation Adoption)

**Why it was required:** `account-management/minime.account.management.system.specification.v1.md` — "Session Policy" described a flat 7-day session with mandatory full re-authentication on expiry, and explicitly stated "V1 does not support: Session Management, Device Lists, Trusted Devices, Logout All Devices." The approved implementation (`implementation/08-security-model.md`, `implementation/05-api-contracts.md` — Part 2) implements and requires exactly the opposite: a 15-minute JWT access token, a 30-day rotating refresh token, `GET /api/v1/auth/sessions` (session listing), and `POST /api/v1/auth/logout-all` (logout all devices). This is a direct, total contradiction of the session model, identified during the Phase A architecture-synchronization audit.

**Resolution:** The architecture is updated to adopt the implemented JWT + refresh token + session management model as canonical. The obsolete flat-session model is removed.

**Why it is NOT an architectural redesign:** The implemented model is already the one described in `implementation/08-security-model.md` (itself derived from standard, already-approved OAuth/OIDC and JWT practice used throughout the Authentication domain) and was already in force for all other authentication documents (`authentication.policy.v1.md` never described a 7-day session either). Only `minime.account.management.system.specification.v1.md` was out of sync. No new Product Domain, no new entity, and no new business capability is introduced — `Session` already exists as entity #3 in `implementation/03-canonical-data-model.md`.

**Supersedes:** The "Session Policy" section previously in `account-management/minime.account.management.system.specification.v1.md`, which stated a flat 7-day session with mandatory re-authentication and explicitly listed "Session Management, Device Lists, Trusted Devices, Logout All Devices" as unsupported. That text is fully replaced, not retained anywhere, since it described a model that was never implemented and would be actively misleading to preserve even as a labeled alternative.

**Scope:** `account-management/minime.account.management.system.specification.v1.md` — "Session Policy" section rewritten to describe: session creation (access + refresh token pair), token model (15-minute JWT access token; 30-day sliding refresh token, rotated on every use, hash-only storage), session expiry/re-authentication trigger, and full session management (session listing, single-session logout, logout-all).

**Explicit Non-Goals:** Does not add device fingerprinting or "trusted device" labeling — V1 treats all sessions as equivalent. Does not change the `Session` entity shape. Does not change any API route already defined in `implementation/05-api-contracts.md`.

**Affected Canonical Documents:**
- `Architecture-Product-Definition/account-management/minime.account.management.system.specification.v1.md` — "Session Policy" — rewritten; this is now the Canonical Owner of the session lifecycle/token-model rule (per `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md` — "Canonical Ownership," corrected in Phase A.5 — G-01)
- `Architecture-Product-Definition/account/authentication.policy.v1.md` — consulted for consistency; not modified (never described a conflicting session model)

**Affected Implementation Documents:** None modified — Implementation was already correct.
- `implementation/08-security-model.md` — "Session Security," "Token Rules" (Operational Specification; reference only)
- `implementation/05-api-contracts.md` — Part 2 (`GET /auth/sessions`, `POST /auth/logout`, `POST /auth/logout-all`) (reference only)

**Dependent Documents:**
- `implementation/03-canonical-data-model.md` — entity 3 (`Session`)
- `implementation/04-service-contracts.md` — `AuthService.createSession`/`refreshSession`/`revokeSession`/`revokeAllSessions`

**Migration Impact:** Documentation-only. Any team member who had read the obsolete "no session management" text and planned frontend work around its absence must update those plans — session list and logout-all are real, buildable V1 endpoints.

**Backward Compatibility:** Fully compatible — no implementation behavior changes.

---

## APD-013 — SEO Indexing Keyed Exclusively on Account.status (No Draft/Publish Workflow)

**Status:** Approved

**Classification:** Architecture Correction (Consistency Fix)

**Why it was required:** `seo/seo.indexing.policy.v1.md` described indexing eligibility in terms of a draft/published/unpublished workflow ("draft profiles," "unpublished profiles," "Published → Draft," "Draft → Published") that does not exist anywhere else in the approved V1 architecture. Every other domain document, and the absolute invariant in `implementation/README.md` ("No Publishing Workflow"), states that Minime V1 is always-live with no draft, unpublished, scheduled, preview, or separate live profile states. This was an internal contradiction within the frozen architecture itself, identified during the Phase A architecture-synchronization audit.

**Resolution:** `seo/seo.indexing.policy.v1.md` is rewritten to key all indexing eligibility, robots directives, sitemap participation, and visibility-change handling exclusively on `Account.status` (`active` / `suspended` / `deleted`), matching the behavior already implemented in `implementation/12-seo-and-integrations.md`.

**Why it is NOT an architectural redesign:** `implementation/12-seo-and-integrations.md` already implements SEO indexing this way; only the architecture document was out of sync. No new status field, no new domain concept, and no new business rule is introduced — `Account.status` already exists as the sole lifecycle field on `Account` (`implementation/03-canonical-data-model.md`, entity 1).

**Supersedes:** The prior text in `seo/seo.indexing.policy.v1.md` referencing "draft profiles," "unpublished profiles," and "Published ↔ Draft" transitions. That text is fully replaced, since it described states that never existed in V1.

**Scope:** `seo/seo.indexing.policy.v1.md` rewritten in full: indexing eligibility, non-indexable conditions, robots policy, canonical policy, and the visibility-change section are all restated in terms of `Account.status` only.

**Explicit Non-Goals:** Does not introduce a draft or publishing capability. Does not change `Account.status`'s enum values or meaning. Does not change robots/canonical/sitemap generation mechanics beyond removing the non-existent draft/publish trigger.

**Affected Canonical Documents:**
- `Architecture-Product-Definition/seo/seo.indexing.policy.v1.md` — rewritten; Canonical Owner of SEO indexing eligibility
- `Architecture-Product-Definition/account/account.model.specification.v1.md` — `Account.status` (reference; not modified, already the sole lifecycle field)
- `Architecture-Product-Definition/public-profile/public-profile.error.states.v1.md` (reference; not modified, already used the same `Account.status` check)

**Affected Implementation Documents:** None modified — Implementation was already correct.
- `implementation/12-seo-and-integrations.md` — "SEO Implementation Rules" (Operational Specification; reference only)

**Dependent Documents:**
- `Architecture-Product-Definition/public-profile/public-profile.error.states.v1.md` (same `Account.status` check backs the public profile 404 decision)

**Migration Impact:** Documentation-only. No engineer needs to change any implementation; the SEO indexing document now matches the always-live model already in force everywhere else.

**Backward Compatibility:** Fully compatible.

---

## APD-014 — Smart Cache Invalidation Adopted for the Application Cache Layer

**Status:** Approved

**Classification:** Architecture Correction (Implementation Adoption)

**Why it was required:** `public-profile/public-profile.cache.policy.v1.md` explicitly listed "Manual Cache Purge" and "Smart Cache Invalidation" as "Not Supported" in V1, stating that profile updates rely solely on TTL expiration. The approved implementation (`implementation/09-caching-strategy.md`) requires immediate invalidation of `profile:public:{username}` and `profile:content:{account_id}` in Redis after every one of eleven distinct profile-mutating operations. This is a direct contradiction identified during the Phase A architecture-synchronization audit, and it materially affects update latency: TTL-only expiration bounds freshness at up to 60 seconds after every edit, while Smart Cache Invalidation of the application cache layer makes most edits visible immediately (bounded only by the independent edge/CDN layer's remaining TTL).

**Resolution:** The architecture is updated to describe Minime's actual two-layer cache design: Layer 1 (Edge/CDN, TTL-only, never explicitly purged) and Layer 2 (Application/Redis, invalidated immediately on mutation). Smart Cache Invalidation is adopted as the official V1 strategy for Layer 2.

**Why it is NOT an architectural redesign:** This is a stronger freshness guarantee than the one previously documented, achieved with a mechanism (`implementation/09-caching-strategy.md` — "Cache Invalidation") that was already fully specified and approved at the implementation layer; only the architecture-level cache policy document had not been updated to reflect it. No new cache technology, no new cache layer, and no new API is introduced. The "Core Principle" that caching never becomes the source of truth and the platform remains correct with or without cache is unchanged and unaffected.

**Supersedes:** The prior "V1 Scope — Not Supported" list in `public-profile/public-profile.cache.policy.v1.md`, which named "Manual Cache Purge" and "Smart Cache Invalidation" together as unsupported. The corrected text separates the two: Manual (CDN-level) purge remains unsupported in V1; Smart Cache Invalidation of the application layer is adopted.

**Scope:** `public-profile/public-profile.cache.policy.v1.md` rewritten to describe the two-layer model, the Smart Cache Invalidation trigger list (by reference to `implementation/09-caching-strategy.md`), and the shared freshness budget mechanism between the two layers.

**Explicit Non-Goals:** Does not introduce explicit Edge/CDN purge — Layer 1 remains TTL-only in V1. Does not change the 60-second maximum combined freshness budget. Does not change cache failure behavior (Redis unavailability must still never block a mutation).

**Affected Canonical Documents:**
- `Architecture-Product-Definition/public-profile/public-profile.cache.policy.v1.md` — rewritten; Canonical Owner of the cache strategy (corrected from listing `implementation/09-caching-strategy.md` as owner during Phase A.5 — G-01)

**Affected Implementation Documents:** None modified — Implementation was already correct.
- `implementation/09-caching-strategy.md` — "Cache Invalidation," "Freshness Budget Propagation" (Operational Specification; reference only)

**Dependent Documents:**
- `implementation/04-service-contracts.md` — every command listed in `09-caching-strategy.md`'s invalidation table

**Migration Impact:** Documentation-only. No infrastructure or CDN configuration changes as a result of this correction — the actual invalidation behavior was already running as implemented; only its architecture-level description changes.

**Backward Compatibility:** Fully compatible.

---

## APD-015 — Account Deletion Cascade Order Promoted From Implementation

**Status:** Approved

**Classification:** Architecture Correction (Implementation Adoption)

**Why it was required:** `account/account.deletion.policy.v1.md` — Section 4 specified an 11-step cascade order (Sessions, Out Links, Analytics Events, Profile Content, Blocks, Appearance State, Connected Accounts, QR Code Records, AI Analysis Sessions, Binary Assets, Account Record Status). The approved implementation (`implementation/10-background-jobs.md` — Job 7) deliberately uses a different order for three specific pairs — Analytics Events before Out Links, Blocks before the Profile Content record, and AI Analysis Sessions before the QR Code record — because the originally-specified order is not safely retryable: a crash and retry partway through the cascade could leave Analytics Events unreachable once their referenced Out Links were already deleted. This divergence was never recorded in this registry, identified during the Phase A architecture-synchronization audit.

**Resolution:** The architecture's cascade order is updated to match the implemented order exactly, with the idempotency rationale documented directly in `account.deletion.policy.v1.md` — Section 4, so the reason for the ordering is visible at its source rather than only in the implementation layer.

**Why it is NOT an architectural redesign:** Every entity's *treatment* (Hard Delete, or status-only retention for the Account record) is unchanged; only the *order* of deletion changes, and only to guarantee that retrying a partially-failed cascade produces the same end state as an uninterrupted run. No entity gains or loses a deletion treatment. Appearance State's status as an embedded JSONB section of Profile Content (not a separate entity) is also clarified in the same update, consistent with `implementation/03-canonical-data-model.md`, which already modeled it this way.

**Supersedes:** The prior 11-step cascade order in `account/account.deletion.policy.v1.md` — Section 4 (Sessions, Out Links, Analytics Events, Profile Content, Blocks, Appearance State, Connected Accounts, QR Code Records, AI Analysis Sessions, Binary Assets, Account Record Status), which was never implemented and is not safely retryable.

**Scope:** `account/account.deletion.policy.v1.md` — Section 4 (Cascade Order), Section 5 (Entity Deletion Rules — Analytics Events, Out Links, Profile Content/Blocks/Appearance State merged into one step, Connected Accounts, AI Analysis Sessions, QR Code Records, Binary Assets), and Section 9 (Ownership Summary table) all updated to the 9-step order: Sessions → Out Link ID Collection (read-only) → Analytics Events → Out Links → Profile Content/Blocks/Binary Assets → Connected Accounts → AI Analysis Sessions → QR Code Record → Account Record Status.

**Explicit Non-Goals:** Does not change any entity's deletion treatment (still Hard Delete throughout, except the Account record itself which remains status-only). Does not change the durability mechanism (`AccountDeletionOutbox`, APD-001) that guarantees this cascade eventually runs. Does not change retention of Audit Logs or the permanently-reserved username, both of which remain outside the cascade.

**Affected Canonical Documents:**
- `Architecture-Product-Definition/account/account.deletion.policy.v1.md` — Sections 4, 5, 9 rewritten; remains the Canonical Owner of the cascade order and now carries the idempotency rationale at its source

**Affected Implementation Documents:** None modified — Implementation was already correct (and was the source of the superior ordering being adopted).
- `implementation/10-background-jobs.md` — Job 7 ("Account Deletion Cascade") (Operational Specification; reference only)
- `implementation/03-canonical-data-model.md` — "Transaction Requirements" (reference only)

**Dependent Documents:**
- APD-001 (`AccountDeletionOutbox`) — the durability guarantee that this cascade is eventually enqueued is unaffected by this reordering

**Migration Impact:** Documentation-only. No change to `implementation/10-background-jobs.md` Job 7 is required — it already implements the now-canonical order. Any future re-implementation of Job 7 from the architecture document alone will now produce the correct, retry-safe order on the first attempt.

**Backward Compatibility:** Fully compatible.

---

## APD-016 — AnalyticsEvent.account_id Snapshot on link_click Rows

**Status:** Approved

**Classification:** Entity (Field Addition)

**Why it was required:** `analytics/analytics.model.specification.v1.md` and `analytics/analytics.data-source.specification.v1.md` define the `link_click` event shape as exactly `{event_id, event_type, out_link_id, created_at}` — no `account_id`. The approved implementation (`implementation/03-canonical-data-model.md` — entity 10, `AnalyticsEvent`) adds `account_id` to `link_click` rows as an immutable snapshot of `OutLink.account_id`, captured at click time, so that account-level click totals (`AnalyticsService.getAnalyticsSummary`) can be computed without joining `OutLinkRepository` (which `AnalyticsService` is architecturally forbidden from depending on) and so that these totals remain correct after the referenced `OutLink` is purged. This addition was never recorded in this registry, identified during the Phase A architecture-synchronization audit.

**Resolution:** The field addition is adopted as approved V1 behavior and recorded here. The architecture documents are not rewritten field-by-field; this entry is the authoritative record that the addition is sanctioned.

**Why it is NOT an architectural redesign:** `account_id` is a read-only snapshot, not a new relationship — `AnalyticsEvent` already conceptually belongs to an Account's activity stream (`analytics/analytics.system.specification.v1.md` — "Analytics Ownership": "Platform Data owns: Account Identity"). This addition only makes an already-implied ownership reference queryable directly, without introducing any new foreign key, index semantics change, or business rule. Analytics remains strictly observational and append-only.

**Supersedes:** Narrows the strict 4-field `link_click` shape stated in `analytics/analytics.model.specification.v1.md` and the 2-field shape stated in `analytics/analytics.data-source.specification.v1.md`; both are additive narrowings (one field added), not contradictions of the events' meaning.

**Scope:** `AnalyticsEvent.account_id` (nullable; required and populated for both `profile_view` and `link_click` event types, per `implementation/03-canonical-data-model.md` — entity 10, "Event shapes"). No foreign key enforcement, consistent with the rest of `AnalyticsEvent`.

**Explicit Non-Goals:** Does not add `account_id` to any other Analytics-owned concept. Does not change `AnalyticsService`'s Forbidden Dependencies (`OutLinkRepository` remains forbidden). Does not enable any new query capability beyond the account-level click total already specified.

**Affected Canonical Documents:**
- `Architecture-Product-Definition/analytics/analytics.model.specification.v1.md` — Link Click Event Model (field list is now understood to include `account_id` as an implementation-level addition per this APD; not directly edited — this APD entry is the authoritative record of the addition, per the Cross Reference Policy in `REPOSITORY_GOVERNANCE.md`)
- `Architecture-Product-Definition/analytics/analytics.data-source.specification.v1.md` — "Out Link Source Data" (same treatment)
- `Architecture-Product-Definition/analytics/analytics.system.specification.v1.md` — "Analytics Ownership" (grounding principle; not modified)

**Affected Implementation Documents:**
- `implementation/03-canonical-data-model.md` — entity 10 (`AnalyticsEvent`), "Event shapes"
- `implementation/04-service-contracts.md` — `AnalyticsService.recordLinkClick`, `getAnalyticsSummary`

**Dependent Documents:** None beyond the Affected documents listed above.

**Migration Impact:** None — the field already exists in the shipped data model; this APD only formalizes its approval status.

**Backward Compatibility:** Fully compatible; the field is additive and nullable, and no existing query or contract is broken by its presence.

---

## Amendment Procedure

Adding a decision to this registry after the date of its creation requires:

1. Confirming the decision is implementation-enabling only (per `implementation/README.md` — "Post-Freeze Clarification Policy") and does not expand product scope, introduce a new Product Domain, or contradict `MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md`.
2. Adding a new `APD-0NN` entry to this document using the **full** Registry Format defined above — including `Supersedes`, `Affected Canonical Documents`, `Affected Implementation Documents`, `Dependent Documents`, `Migration Impact`, and `Backward Compatibility`. An entry missing any of these fields is incomplete and fails the APD Validation check in `REPOSITORY_GOVERNANCE.md` — "Architecture Drift Prevention."
3. Confirming the entry's **Affected Canonical Documents** field names at least one Architecture-tier document, or explicitly states "None directly" with a grounding principle cited — an APD must never leave its architectural grounding unstated.
4. If the decision corrects prior text in an Architecture document (rather than adding something net-new), editing that document directly in the same change, and recording what was superseded in the `Supersedes` field — never leaving the correction to exist only inside this registry entry while the Architecture document itself remains unchanged and contradictory.
5. Updating `implementation/README.md` and `Architecture-Product-Definition/docs/MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md` only if the amendment changes what those documents already state about this registry's existence — never duplicating the decision's content into either document.

No decision is valid until it is recorded here with every required field populated.
