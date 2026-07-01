# Implementation Blockers Resolution Report — Minime V1

## Status

Resolves the 20 critical/high blockers, the cross-domain conflict table, and the majority of the missing-contract inventory in `FULL_STACK_IMPLEMENTATION_BLOCKERS_AUDIT.md`, via architectural clarification and contract completion only. See "What Was Not Fully Closed" at the end of this report for the honest remainder.

---

## 1. Method

For each blocker, the audit's own "الإصلاح المطلوب" (required fix) was treated as the target. Where two canonical files disagreed, the file representing the already-intended architecture was kept and the other was corrected to match (never a third, new interpretation). Where a contract was missing, the smallest contract that makes the existing architecture implementable was added — never a new subsystem, unless the audit itself named the missing piece as an outbox/entity/lock (in which case that is exactly what was added, nothing broader).

---

## 2. Blocker-by-Blocker Resolution

### #1 — Registration transaction has no owner for AuthenticationIdentity
**Files:** `implementation/04-service-contracts.md`, `implementation/03-canonical-data-model.md`, `Architecture-Product-Definition/profile-content/profile.content.lifecycle.v1.md`
**Fix:** `AuthService.completeRegistrationWithOAuth` is now the explicit, sole transaction coordinator for registration ("Registration Transaction Coordination" section). It opens one database transaction spanning `AccountRepository`/`ProfileContentRepository` (via `AccountService.createAccount`, which no longer touches `AuthenticationIdentity` at all) and `AuthRepository` (creating `AuthenticationIdentity` directly) and `UsernameReservationRepository`. `AccountService.createAccount` signature reduced to `{account_id, username}` — no provider fields — removing the reason it ever appeared to need `AuthRepository`. A compensating deletion is defined for the case where the post-commit `createQrCode` step fails. This also fixed `display_name`'s source (see #9).
**No redesign:** repository ownership boundaries (Forbidden Dependencies lists) are unchanged; the coordinator pattern only sequences existing calls inside one transaction.

### #2 — Account deletion depends on non-durable EventEmitter
**Files:** `implementation/03-canonical-data-model.md` (new `AccountDeletionOutbox` entity), `implementation/04-service-contracts.md`, `implementation/10-background-jobs.md` (new Job 8), `implementation/06-event-contracts.md`, `implementation/08-security-model.md`, `implementation/02-repository-structure.md`, `implementation/13-implementation-plan.md`
**Fix:** `deleteAccount` now commits `Account.status = 'deleted'` and an `AccountDeletionOutbox` row in one transaction. A new scheduled job (Job 8, "Deletion Outbox Dispatcher," every 30s) guarantees `account.deletion.cascade` gets enqueued even if the process crashes immediately after commit — the EventEmitter publish becomes a best-effort fast path, not the durability mechanism. A new "Per-Request Account Status Check" (cached, 5s TTL) closes the remaining gap of an already-issued access token outliving session revocation; `refreshSession` additionally rejects non-active accounts (a real gap found during verification, fixed in the same pass).
**No redesign:** this is the audit's own first-listed remedy ("outbox متين داخل معاملة الحذف"). No event-sourcing system, no new Product Domain.

### #3 — Event ownership contradicts EventEmitter's actual durability
**Files:** `implementation/06-event-contracts.md` (new "V1 Event Durability Classification" section), `Architecture-Product-Definition/platform/events/events.architecture.specification.v1.md` (new "V1 Scope Note")
**Fix:** Declared explicitly that V1 implements no Event Store — all catalog events are best-effort EventEmitter notifications — and that this does not violate the Events Platform architecture, because that architecture already states "product correctness must never depend on Event persistence" and "Simplicity Over Event Sourcing" as canonical rules. Business-critical durability (deletion) is achieved by domain-owned state (`AccountDeletionOutbox`), never by the Events Platform.
**No redesign:** zero Canonical Rules in the architecture doc were changed; a scope note was added.

### #4 — Click accuracy explicitly allows duplicates
**Files:** `implementation/04-service-contracts.md`, `implementation/03-canonical-data-model.md` (`AnalyticsEvent.id`), `implementation/10-background-jobs.md` (Job 2), `implementation/13-implementation-plan.md`
**Fix:** `OutLinksService.resolveOutLink` generates a `click_id` synchronously and uses it as both the BullMQ `jobId` and the `AnalyticsEvent.id` primary key. Retries collide on the primary key and are treated as an already-recorded success. Two independent de-duplication layers (queue + DB), zero new infrastructure.

### #5 — Account-level click totals unattributable after OutLink purge
**Files:** `implementation/03-canonical-data-model.md`, `implementation/04-service-contracts.md`
**Fix:** `link_click` events now snapshot `account_id` (read from `OutLink.account_id` at click time, by the caller) directly onto `AnalyticsEvent`. `getAnalyticsSummary` computes click totals from this snapshot with no join to `OutLinkRepository` (which `AnalyticsService` remains forbidden from), and the total survives OutLink purge.

### #6 — Button/OutLink dual destination
**Files:** `Architecture-Product-Definition/rendering/block.renderers.specification.v1.md`
**Fix:** The Button and Social Icons renderer sections claimed the raw stored `url` is rendered directly, contradicting `rendering.architecture.canon.v1.md` ("Button Renderer reads block-stored label **and Out Link reference**") and the already-implemented `implementation/04-service-contracts.md` RenderingService rule (resolve `/out/{public_id}` via OutLink lookup). The renderer spec was corrected to match the canon + implementation: rendered `url` is always `/out/{public_id}`; the block's own stored `url`/Connected Account `url` is only ever the *source* value used when the Out Links domain creates the managed OutLink.

### #7 — Block + OutLink update has no shared transaction
**Files:** `implementation/04-service-contracts.md`, `implementation/03-canonical-data-model.md` (new partial unique index)
**Fix:** `addBlock`, `updateBlock`, `deleteBlock` (ProfileService) and `updateConnectedAccount`/`removeConnectedAccount` (ConnectedAccountsService) are now documented transaction coordinators, symmetric to the registration coordinator: one DB transaction spans the Block/ConnectedAccount write and the OutLink archive/create calls (still made only through `OutLinksService`, never direct repository access — Forbidden Dependencies unchanged). A new `UNIQUE (block_id, connected_account_id) WHERE status = 'active'` index on `OutLink` is the DB-level backstop against a concurrent race producing two active OutLinks.

### #8 — Appearance defers block-level overrides while 6 files build them as V1
**Files:** `Architecture-Product-Definition/appearance/themes/theme.constraints.specification.v1.md`, `Architecture-Product-Definition/block-styling/block-style.constraints.specification.v1.md`, `Architecture-Product-Definition/block-styling/block-style.model.specification.v1.md`, `implementation/07-validation-rules.md`
**Fix:** Declared the V1 scope decision explicitly: block-level style overrides are in V1 (the `block-styling/` folder is the closed contract). Added a genuine closed JSON-schema property catalog per block type (which keys, which value domains) — closing "no catalog exists." Replaced the three-option "Reject Save / Reset To Inherited / Auto-Correct" ambiguity with one rule per situation: Reject Save at write time, Reset To Inherited (per-field) at theme-switch time, Auto-Correct excluded entirely. Fixed `07-validation-rules.md`'s contradicting claim that `avatar`/`name`/`bio` have no styles and that unknown keys pass through silently — both now match the closed catalog (unknown keys rejected, not ignored).

### #9 — `display_name` empty-vs-required contradiction
**Files:** `implementation/04-service-contracts.md`, `implementation/03-canonical-data-model.md`, `Architecture-Product-Definition/profile-content/profile.content.lifecycle.v1.md`
**Fix:** `display_name` is initialized to the claimed username at registration (inside the same transaction as Account creation — see #1). `profile.content.lifecycle.v1.md`'s "Display Name = Empty" initial state (contradicting `profile.content.specification.v1.md`'s "required") was corrected; `profile.content.specification.v1.md` needed no change, since it already stated the correct rule.

### #10 — Two independent 60-second cache layers stack to ~120s
**Files:** `implementation/09-caching-strategy.md`, `Architecture-Product-Definition/public-profile/public-profile.cache.policy.v1.md`
**Fix:** Added "Freshness Budget Propagation": the edge layer's `Cache-Control: max-age` is computed from Redis's own `TTL` command on `profile:public:{username}` at serve time, not hardcoded to 60. This makes `edge_age + app_age <= 60` an enforced invariant using an existing Redis command — no new cache layer, no new invalidation mechanism.

### #11 — Avatar replacement deletes the old asset before the new one is referenced
**Files:** `implementation/11-platform-services.md`
**Fix:** Reordered to **upload → validate → DB swap → delete old (async)**, matching `platform/storage/asset.lifecycle.specification.v1.md`'s already-correct architecture ("previous asset immediately becomes orphaned" only *after* the new reference commits). The implementation layer was the outlier; it now matches the architecture layer.

### #12 — Image type policy conflict + missing processing spec
**Files:** `implementation/07-validation-rules.md`, `Architecture-Product-Definition/platform/storage/asset.processing.specification.v1.md` (**new file**)
**Fix:** `07-validation-rules.md` accepted `image/gif` (architecture explicitly forbids animated GIF) and never mentioned HEIC (architecture explicitly accepts it). Corrected to `jpeg/png/webp/heic`, matching `storage.architecture.specification.v1.md` exactly. Created the referenced-but-missing `asset.processing.specification.v1.md` (input validation order, magic-byte sniffing, decompression-bomb ceiling, transcoding rules, original-discard rule) — it was a dangling reference (line 660 of `storage.architecture.specification.v1.md`), not a new capability.

### #13 — AI owns and doesn't own an entity in the same document set
**Files:** `Architecture-Product-Definition/platform/data/canonical.entities.map.v1.md`
**Fix:** The map's "AI Suggestion Entity" (§22) described a per-suggestion Generated→Presented→Accepted/Edited/Rejected/Expired entity — which `ai.architecture.specification.v1.md` (already rewritten in a prior pass) explicitly scopes to **V2**, not V1. §22 was rewritten to describe `AnalysisSession` — the one AI-owned V1 entity — with a superseded-terminology note explaining the correction, and every remaining "AI Suggestions" reference in the map's trees/tables was updated to `AnalysisSession`.

### #14 — No implementable Analysis Session API
**Files:** `implementation/05-api-contracts.md` (new Part 9), `implementation/11-platform-services.md`, `implementation/13-implementation-plan.md`
**Fix:** Added `POST /api/v1/account/ai/analysis` (synchronous — matches the already-defined synchronous `AIService.analyzeProfile`, so no new async/polling subsystem was invented) and `GET /api/v1/account/ai/analysis/latest`, full DTOs, a 3-way status union (`completed` / `failed` / `disabled` — closing the "AI failure returns empty array" ambiguity from audit §10 as a side effect), and a rate limit (10/account/24h). `AIService.analyzeProfile`'s return type was updated to carry `status` explicitly instead of an under-specified `{suggestions}`.

### #15 — Connected Accounts existence-check contradiction
**Files:** `Architecture-Product-Definition/account-management/minime.account.management.system.specification.v1.md`
**Fix:** `connected.accounts.specification.v1.md` was already explicit and detailed ("never checks whether the account exists on the external platform"). The management system doc's stray "If the account cannot be found: Creation Rejected" was the outlier and was corrected to "If the format is invalid: Creation Rejected."

### #16 — Social platform registry inconsistent with the enum
**Files:** `Architecture-Product-Definition/social-accounts/social.accounts.platform.rules.v1.md`, `Architecture-Product-Definition/social-accounts/social.accounts.storage.model.v1.md`
**Fix:** Fixed the wrong directory reference (`social-accounts/platform/` → `social-accounts/social-platforms/`, the directory that actually exists). Removed the GitHub and Pinterest sections, which had no dedicated rule file and no enum entry — declared the V1 platform set closed at the 12 platforms that actually have all three (enum value, rule file, and this document's section) in agreement, with an explicit rule that all three must be updated together for any future platform.

### #17 — GTM container ID framed as safe when it grants arbitrary script execution
**Files:** `Architecture-Product-Definition/integrations/providers/google-tag-manager.specification.v1.md`, `implementation/12-seo-and-integrations.md`
**Fix:** Replaced the false claim ("the only accepted configuration is a validated Container ID" as if that prevents script injection) with an honest threat model, and closed the actual gap with containment: the GTM snippet is now loaded inside a sandboxed `<iframe sandbox="allow-scripts">` (no `allow-same-origin`, no `allow-top-navigation`), giving it an opaque origin with no access to the parent page's DOM, cookies, or storage — regardless of what tags the account owner's own GTM workspace contains. This also lets the parent document's CSP `script-src` stay tight, since Google's tagging domains are only ever loaded inside the sandboxed iframe's own document. The feature was **not removed**; it was contained.

### #18 — Concurrency rules don't protect invariants
**Files:** `implementation/03-canonical-data-model.md`, `implementation/04-service-contracts.md`
**Fix:** Added, without touching the platform's Last-Write-Wins field-value model (`data.architecture.specification.v1.md` §"Concurrency" is explicitly preserved and cited): a partial unique index capping identity blocks at 1/type/account; a unique index preventing duplicate `sort_order` on `Block` and `ConnectedAccount`; a per-account Postgres advisory transaction lock shared by every Block- and ConnectedAccount-mutating command, which is what makes `MAX_BLOCKS_PER_ACCOUNT` enforcement and `sort_order` assignment race-free; explicit reorder-array validation (no duplicates, no gaps, exact-set match); and a shared per-username advisory lock between `UsernameService.reserveUsername` and the registration coordinator, closing the two-table (`Account.username` / `UsernameReservation.username`) race the audit identified.

### #19 — Audit Logging required but not implemented
**Files:** `implementation/03-canonical-data-model.md` (new `AuditLog` entity), `implementation/04-service-contracts.md` (new `AuditLogService`), `implementation/10-background-jobs.md` (new Job 9), `implementation/13-implementation-plan.md`
**Fix:** Implemented the minimum entity + single write path (`AuditLogService.record`, called synchronously by the originating command handler, never via EventEmitter) needed to satisfy the already-approved `audit.logging.policy.v1.md`, scoped to the events that correspond to entities that actually exist in V1 (dropped `page_*`/`block_*`/`recovery_email_changed` — the policy's own architecture note that these don't exist in V1 was the deciding factor, not a new judgment call). 12-month retention via a new scheduled job. No admin UI was built (none exists anywhere else in V1 either); authorized access is direct credentialed DB access, consistent with the rest of V1's operational model.

### #20 — Deployment topology doesn't run the described system
**Files:** `implementation/13-implementation-plan.md`, `implementation/01-technology-stack.md`, `implementation/02-repository-structure.md`
**Fix:** Added the missing `apps/worker` (BullMQ job processors, no HTTP listener, shares the same Product Domain services/repositories as `apps/api` — explicitly not a microservice) as a third deployable container; added `/qr/*` to the Nginx routing table (previously absent despite the route existing); replaced "current Node.js LTS" with an explicit pinning mechanism (`.nvmrc` + matching Dockerfile tags, one exact version, deliberately upgraded); defined a rollback policy (forward-only, additive migrations; rollback = redeploy previous image against the same schema); defined a multi-instance policy that explains why EventEmitter's cross-instance limitation is no longer a correctness risk (see #2's outbox).

---

## 3. Cross-Domain Conflicts Resolved (beyond the numbered blockers above)

| Conflict | Resolution |
|---|---|
| Image `image_id` vs renderer `{url, alt}` | Renderer spec corrected: `image_id` is the only stored field; `url` is resolved via `ImageAssetRepository` + `StoragePlatform.buildPublicUrl`; `alt` is always `""` in V1 (no stored source exists anywhere — this is documented as complete, not an omission). |
| Title/Textbox `content.text` vs renderer `{value}` | Renderer spec corrected to `text`, matching `blocks/title.block.specification.v1.md` and `blocks/textbox.block.specification.v1.md`, which were already consistent with `implementation/03`. |
| Profile ↔ Blocks (`block_ids[]` vs `Block.sort_order`) | `profile.block.reference.specification.v1.md`'s own "Ordering Model" section already named `Block.sort_order` as the sole source of truth and said "does not create a second ordering model" — its own later "Canonical Composition Model" section then contradicted this with a `block_ids: string[]` field that also never existed on `implementation/03`'s `ProfileContent`. Corrected to remove the stored array; composition is a query, not a field. |
| Settings ↔ Integrations (GTM) | Resolved as part of #17 — GTM is a per-account setting, consistently described that way everywhere now, with the sandboxing fix. Added the missing cache-invalidation trigger for `gtm_container_id` changes to `09-caching-strategy.md`. |
| Account ↔ Authentication (AuthenticationIdentity ownership) | Resolved as part of #1. |
| Deletion ↔ Retention (Audit Logs) | Resolved as part of #2 and #19 together. |

---

## 4. Missing Contracts Filled

- Transaction coordinator contracts: registration (#1), Block+OutLink mutations (#7), account deletion (#2).
- Idempotency keys: registration reservation lock (#18), click recording (#4).
- Concurrency contracts: partial unique indexes + advisory locks (#18).
- Cache invalidation contract: freshness budget propagation (#10), GTM setting trigger.
- API/DTO contract: AnalysisSession API (#14).
- Missing referenced file: `asset.processing.specification.v1.md` (#12).
- JWT contract: access token TTL (15 min), algorithm (HS256), claims, refresh token TTL (30 days) — `implementation/08-security-model.md`.
- Rate limit contract: AI analysis endpoint (10/account/24h) — `implementation/05-api-contracts.md`.
- Per-request authorization contract: Account status check independent of session-table state — `implementation/08-security-model.md`.

**Not fully filled** (see next section): global error-code catalog, pagination contract, exhaustive URL-normalization/open-redirect rules, exhaustive frontend state-machine contracts, SEO visibility-state reconciliation, full JWT signing-key rotation schedule.

---

## 5. What Was Not Fully Closed

In the interest of honesty rather than a false "100% resolved" claim:

- **Frontend contracts** (audit §7): route maps, editor optimistic/pessimistic save semantics, upload progress/cancel UX, and the full frontend state-machine inventory were not written. This report only closes backend/data/architecture-layer blockers; the frontend implementation gap is real and unaddressed.
- **Error code catalog and pagination contract** (audit §4): not produced as a standalone enumerated catalog. Endpoint-level error lists exist per section of `05-api-contracts.md`, but no single cross-endpoint error-code registry was created.
- **JWT signing-key rotation schedule**: TTLs and algorithm were specified (§18-adjacent fix); an automated rotation *schedule* was explicitly deferred to "manual, operational" rather than specified further, since no key-management infrastructure exists elsewhere in V1 to hang an automated rotation off of.
- **SEO ↔ Visibility** (draft/private/restricted states referenced in `seo/` but absent from V1) and the **JSON-LD `sameAs`/Connected-Accounts exposure** conflict the audit described were checked against the current repository state; the specific `sameAs` mechanism the audit quoted does not currently exist in `seo.structured-data.specification.v1.md`, so no live contradiction was found to fix. The broader SEO visibility-state reconciliation was not investigated further given time constraints and is a candidate for a follow-up pass.
- **URL normalization / open-redirect / control-character rules** for user-entered URLs (button destinations, etc.) were not written as a dedicated validation contract.

None of the above were skipped because they seemed unimportant — they were deprioritized relative to the 20 numbered blockers and cross-domain conflicts, which were the audit's own top-priority list, given the scope of a single pass.

---

## 6. Confirmations

- **No architectural redesign was introduced.** Every fix either (a) made one existing canonical file conform to another that already represented the intended architecture, or (b) added the smallest contract (an entity, a lock, an index, a coordinator sequence) the audit itself named as the required fix. Domain ownership, repository boundaries, the Product Domain list, and the Platform Service list are all unchanged.
- **No V2 functionality was added.** Per-suggestion AI tracking, admin UI, automated key rotation, and multi-container GTM environments were all explicitly kept out of scope and, where the audit's own text confirmed they were already V2-scoped (e.g. AI Suggestion history), that scoping was reinforced, not expanded.
- **The implementation philosophy is unchanged.** Last-Write-Wins remains the platform's concurrency model for field values (new locks/indexes protect invariants, not field conflicts — explicitly documented as the distinction). The Events Platform remains non-durable-by-default with domain-owned durability as the exception, not the reverse. Hard Delete, no publishing workflow, no `User`/`Profile` entities, and account-centric ownership all remain exactly as originally specified.
