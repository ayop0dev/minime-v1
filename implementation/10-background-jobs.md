# Minime V1 — Background Jobs

## Status

Canonical. Final.

---

## Architecture Authority

```
Architecture-Product-Definition/account/authentication.policy.v1.md
Architecture-Product-Definition/account/account.deletion.policy.v1.md
Architecture-Product-Definition/out-links/out-link.system.specification.v1.md
Architecture-Product-Definition/analytics/analytics.model.specification.v1.md
Architecture-Product-Definition/platform/data/data.retention.policy.v1.md
implementation/03-canonical-data-model.md
implementation/04-service-contracts.md
implementation/06-event-contracts.md
```

---

## Purpose

Defines the canonical background job model for Minime V1.

Background jobs execute outside the HTTP request lifecycle.

Background jobs define:

- job ownership
- trigger
- payload
- execution behavior
- retry and idempotency rules
- scheduling (where applicable)

---

## Technology

**BullMQ** is the only approved background job system for Minime V1.

BullMQ is backed by Redis. BullMQ owns retry logic, scheduling, and job state.

Background jobs must not execute directly from HTTP controllers.

Business state belongs to PostgreSQL. Job state belongs to BullMQ. Jobs must not become the source of truth.

---

## Global Job Rules

### Creation

- Background jobs may be created only after successful business persistence.
- Jobs must not be created before the transaction commits.
- Jobs represent completed business operations, not intentions.

### Payload

- Job payloads must contain only the information required for execution.
- Payloads must not contain passwords, access tokens, refresh tokens, or secrets.
- Large business entities must not be embedded in payloads. Reference entities by identifier.

### Idempotency

Every background job must be idempotent. Repeated execution of the same job must produce the same business result. Duplicate execution must not create duplicate entities, publish duplicate events, or produce inconsistent business state.

### Retry

- Failed jobs may be retried per BullMQ retry policy.
- Retry logic belongs to BullMQ, not to EventEmitter.
- Job failures must not corrupt business data.
- Business persistence must not be rolled back because of asynchronous job failures.

### Timeout

Every job must define a maximum execution time. Jobs exceeding the configured timeout must fail. Timed-out jobs may be retried per retry policy.

### Security

- Jobs execute with backend permissions.
- Jobs must validate business ownership before mutating business entities.
- Jobs must not bypass Product Domain validation.
- Jobs must not access HTTP request state.

---

## Canonical Job Catalog

The following jobs are the only V1-approved background jobs. Each is traceable to the frozen architecture.

**Note on numbering:** Job 1 (OTP Verification Expiry Cleanup) was removed when Email OTP authentication was replaced by the External Identity Provider model. Jobs are numbered starting at 2 to preserve referential stability.

### 2. OutLink Click Recording

**Queue:** `out-links.click`

**Owner:** Out Links domain / Analytics domain

**Trigger:** Enqueued by `OutLinksService.resolveOutLink` with `jobId = click_id` (see `04-service-contracts.md`), before the HTTP 302 response is sent. The HTTP redirect must not wait for this job to execute.

**Payload:**

```json
{ "click_id": "uuid", "out_link_id": "uuid", "account_id": "uuid" }
```

**Behavior:**

- Calls `AnalyticsService.recordLinkClick({ click_id, out_link_id, account_id })`.
- Inserts `AnalyticsEvent { id: click_id, event_type: 'link_click', out_link_id, account_id, created_at }`.
- After persistence (including the "already recorded" case below), publishes `out-link.clicked` via EventEmitter.

**Counting point:** A click is counted exactly once, at the moment this job successfully inserts (or confirms as already inserted) the `AnalyticsEvent` row. This is independent of whether the HTTP client ever receives the 302 response — a client disconnecting before the response arrives does not affect the count, since the job was already enqueued with a fixed `click_id` at resolve time.

**Retry:** Yes. `jobId = click_id` gives BullMQ its own de-duplication at the queue level; the `AnalyticsEvent.id = click_id` primary key gives a second, database-level de-duplication layer independent of the queue. On retry, the insert either succeeds (first attempt) or conflicts on `id` (subsequent attempts), which `AnalyticsService.recordLinkClick` treats as an already-recorded success — never a duplicate row, never a duplicate count.

**Why BullMQ:** Click recording must not block the public redirect. The HTTP 302 response is returned immediately. Analytics recording executes asynchronously after the redirect.

---

### 3. UsernameReservation Expiry Cleanup

**Queue:** `username.reservation.expire` (scheduled)

**Owner:** Username domain

**Trigger:** Scheduled — runs periodically (e.g., every 5 minutes).

**Payload:** None (scheduled sweep), or individual `{ "reservation_id": "uuid" }` when triggered for a specific record.

**Behavior:**

- Calls `UsernameService.expireReservation({ id })` for each expired record.
- Hard-deletes `UsernameReservation` records where `expires_at < NOW()` and no Account has been created from them.
- Publishes `username.reservation.expired` event.

**Retry:** Yes. Idempotent.

**Architecture basis:** `04-service-contracts.md` — `expireReservation` is called by background job only; no HTTP endpoint.

---

### 4. Session Cleanup

**Queue:** `auth.session.cleanup` (scheduled)

**Owner:** Authentication domain

**Trigger:** Scheduled — runs periodically (e.g., daily).

**Payload:** None (scheduled sweep).

**Behavior:**

- Removes `Session` records where `revoked_at IS NOT NULL` OR `expires_at < NOW() - [retention buffer]`.
- Session records are safe to remove once they can no longer produce valid tokens.

**Retry:** Yes. Idempotent.

**Notes:** This is housekeeping only. Revoked and expired sessions are already rejected at authentication time regardless of whether the record is physically present.

---

### 5. OutLink Purge (Archived → Hard Delete)

**Queue:** `out-links.purge` (scheduled)

**Owner:** Out Links domain

**Trigger:** Scheduled — runs periodically (e.g., daily).

**Payload:** None (scheduled sweep).

**Behavior:**

- Calls `OutLinksService.purgeArchivedOutLinks()`.
- Hard-deletes `OutLink` records where `status = 'archived'` AND `archived_at < NOW() - 90 days`.
- Records in `created` or `active` status must not be touched.

**Retry:** Yes. Idempotent.

**Architecture basis:** `out-link.system.specification.v1.md` — archived retention: 90 days, then purged. `03-canonical-data-model.md` — OutLink lifecycle.

---

### 6. AnalyticsEvent Retention

**Queue:** `analytics.retention` (scheduled)

**Owner:** Analytics domain

**Trigger:** Scheduled — runs periodically (e.g., daily or weekly).

**Payload:** None (scheduled sweep).

**Behavior:**

- Calls `AnalyticsService.purgeOldEvents()`.
- Hard-deletes `AnalyticsEvent` records past the configured retention period, acting on `created_at`.
- The specific retention duration is an implementation configuration value; the architecture does not define an exact period.
- Publishes `analytics.retention.completed` after completion.

**Retry:** Yes. Idempotent (safe to re-run; already-deleted records are simply not found again).

**Architecture basis:** `data.retention.policy.v1.md` — analytics events may expire or be deleted. `analytics.model.specification.v1.md` — events are append-only; deletion is by retention job only.

---

## Job Prohibitions

Implementation must not introduce jobs for:

- **Publishing** — no publishing workflow in V1
- **Social/Connected Accounts OAuth token refresh** — Connected Accounts are lightweight, user-declared social links, not OAuth connections; no such tokens exist to refresh. (This is distinct from Authentication's Google/Apple sign-in OAuth, which exists but persists no provider tokens — see `08-security-model.md`, so there is nothing for a job to refresh there either.)
- **Social account verification** — Social Accounts domain owns no storage and has no verification flow
- **Email OTP delivery** — Email OTP is not supported in V1; authentication is provider-based
- **Phone OTP delivery** — phone OTP is not supported in V1
- **SMS delivery** — SMS is not supported in V1
- **Profile publication state management** — no draft/published state in V1
- **ConnectedAccount OAuth sync** — ConnectedAccounts are lightweight social links, not OAuth connections
- **Cache warming as a required operation** — cache warming is optional; application correctness must not depend on it

Implementation must not:

- execute background jobs inside HTTP controllers
- use jobs as the source of truth
- persist business state inside BullMQ
- bypass Product Domain services in job workers
- bypass repositories in job workers
- publish success events before successful persistence
- depend on job execution order for business correctness

---

## Job Flow

Every asynchronous operation follows this flow:

```
Business Operation
    │
    ▼
Repository Persistence
    │
    ▼
Transaction Commits
    │
    ▼
Queue BullMQ Job
    │
    ▼
BullMQ Worker Executes (async, with retry)
    │
    ▼
Service
    │
    ▼
Repository → PostgreSQL
```

Job workers must go through Product Domain services, not directly to repositories or Prisma.

---

### 7. Account Deletion Cascade

**Queue:** `account.deletion.cascade`

**Owner:** Account domain (orchestration only; execution delegates to owning domain services)

**Trigger:** Enqueued with `jobId = "account-deletion-cascade:{account_id}"`. Two independent paths may enqueue it: (a) the best-effort `account.deleted` EventEmitter handler, as a fast path; (b) the Deletion Outbox Dispatcher (Job 8), which is the durability guarantee — it enqueues this job within 30 seconds even if path (a) never runs because the process crashed. `jobId` deduplication makes double-enqueue from both paths a safe no-op.

**Payload:**

```json
{ "account_id": "uuid" }
```

**Behavior:**

Executes cross-domain cascade cleanup for a deleted account. Before executing, verifies `Account.status = 'deleted'` via AccountRepository (safety check; aborts if account is not in deleted state).

Cascade steps in order:

1. `AuthService.revokeAllSessions({ account_id })` — catches any sessions not revoked by the synchronous EventEmitter handler
2. `OutLinksService.getAccountOutLinkIds({ account_id })` — read-only; collects all OutLink IDs (any status) BEFORE any deletion; stored in job working memory for step 3
3. `AnalyticsService.deleteAccountAnalytics({ account_id, out_link_ids })` — hard-deletes profile_view events for account_id and link_click events for the out_link_ids collected in step 2; runs BEFORE OutLink deletion to guarantee idempotency
4. `OutLinksService.deleteAccountOutLinks({ account_id })` — hard-deletes all OutLink records regardless of status
5. `ProfileService.deleteAccountData({ account_id })` — hard-deletes all Blocks; hard-deletes all ImageAssets (calls StoragePlatform.delete(storage_key) for each); hard-deletes ProfileContent (includes appearance_config JSONB); calls StoragePlatform.delete(avatar_storage_key) if set
6. `ConnectedAccountsService.deleteAccountConnectedAccounts({ account_id })` — hard-deletes all ConnectedAccount records
7. `AIService.deleteAccountAnalysisSessions({ account_id })` — hard-deletes all AnalysisSession records for the account regardless of status (pending, completed, failed)
8. `AccountService.deleteQrCode({ account_id })` — if an `AccountQRCode` record exists: calls StoragePlatform.delete(canonical_asset_id) to remove the QR SVG asset, then hard-deletes the `AccountQRCode` record. After this step, `GET /qr/{qr_code_id}` no longer resolves to an active profile.

Notes:
- AppearanceState is not a separate entity; appearance_config is a JSONB field on ProfileContent, deleted in step 5.
- There is no `AiDecision`, `AcceptedDecision`, or `SuggestionHistory` entity in V1. `AnalysisSession` is the only AI-owned entity; it is hard-deleted in step 7 regardless of status.
- `AccountQRCode` is a separate Account-owned entity (entity 12 in `03-canonical-data-model.md`), not a field on Account. Its deletion in step 8 is a record deletion, not just an asset deletion.
- StoragePlatform.delete is idempotent — deleting a non-existent key does not throw.
- ConnectedAccount deletion uses Hard Delete (no soft delete, no deleted_at).
- Account record is retained permanently; only the `AccountQRCode` record and its SVG asset are removed.

**Idempotency on retry:** If any step fails and the job retries:
- Step 2 re-reads OutLink IDs. If OutLinks were already deleted (step 4 completed in a prior attempt), step 2 returns an empty list. Step 3 will only clean up profile_view events (link_click events were already cleaned up in the prior attempt). Steps 3 and 4 are both idempotent no-ops if their target records no longer exist.
- All other steps are idempotent: hard-deleting already-deleted records produces no error; StoragePlatform.delete on an already-deleted key does not throw.

**Partial failure behavior:** If any step fails, the job fails and retries from the beginning. Account.status = 'deleted' remains set throughout all retries.

**Retry:** Yes. Every step must be idempotent.

**No "account deletion cancellation" job exists.** Deletion is final in V1.

**Architecture basis:** `account.deletion.policy.v1.md` — cascade ownership rules. `04-service-contracts.md` — domain service deletion commands. `03-canonical-data-model.md` — account deletion cascade.

---

### 8. Deletion Outbox Dispatcher

**Queue:** `account.deletion.outbox.dispatch` (scheduled)

**Owner:** Account domain

**Trigger:** Scheduled — runs every 30 seconds.

**Payload:** None (scheduled sweep).

**Behavior:**

This job is the durability guarantee for account deletion. It exists because `account.deleted` EventEmitter delivery is best-effort and non-durable (see `06-event-contracts.md`), and per `events.architecture.specification.v1.md` — "Failure Philosophy" — business correctness must never depend solely on Event persistence.

1. `SELECT * FROM AccountDeletionOutbox WHERE dispatched_at IS NULL` (see `03-canonical-data-model.md` — AccountDeletionOutbox).
2. For each row: enqueue `account.deletion.cascade` (Job 7) with a deterministic `jobId = "account-deletion-cascade:{account_id}"`. BullMQ deduplicates on `jobId`, so this is a safe no-op if the best-effort EventEmitter path already enqueued the same job.
3. Set `dispatched_at = NOW()` and increment `attempts` on the outbox row.

**Retry:** Yes. Idempotent — re-dispatching an already-enqueued `account.deletion.cascade` job is a no-op by construction (step 2).

**Reconciliation:** A row with `dispatched_at IS NULL` and `requested_at` older than 1 hour indicates this job itself is failing systemically (not an individual account problem) and must page an operator. This is the sole reconciliation mechanism for stuck deletions; no separate monitoring subsystem exists.

**Architecture basis:** `Architecture-Product-Definition/platform/events/events.architecture.specification.v1.md` — "Failure Philosophy," `event.lifecycle.specification.v1.md` — "Product correctness always has higher priority than Event recording." `03-canonical-data-model.md` — AccountDeletionOutbox.

---

### 9. Audit Log Retention

**Queue:** `audit-log.retention` (scheduled)

**Owner:** Platform

**Trigger:** Scheduled — runs daily.

**Payload:** None (scheduled sweep).

**Behavior:**

- Hard-deletes `AuditLog` records where `created_at < NOW() - 12 months`.
- No archival tier exists in V1, per `audit.logging.policy.v1.md` — "Expiration."

**Retry:** Yes. Idempotent.

**Architecture basis:** `Architecture-Product-Definition/account-management/audit.logging.policy.v1.md` — Retention Policy.

---

## Account Deletion Cascade Summary

Account deletion executes in three phases:

**Synchronous (within the HTTP request, before response returns):**
1. `Account.status = 'deleted'` and an `AccountDeletionOutbox` row are committed atomically in one transaction — AccountService
2. After commit: `AuditLogService.record(...)` writes the `account_deleted` audit row, then `account.deleted` is published via EventEmitter as a best-effort notification (cache invalidation and any session revocation that happens to succeed synchronously — neither is relied upon for correctness)

**Durably guaranteed (Deletion Outbox Dispatcher, Job 8, within 30 seconds even in a total-crash scenario):**
3. `account.deletion.cascade` (Job 7) is enqueued — guaranteed by the `AccountDeletionOutbox` row from step 1, independent of whether the EventEmitter publish in step 2 ever ran.

**Asynchronous (account.deletion.cascade BullMQ job, Job 7):**
4–11. Cross-domain cascade as defined in Job 7 above: sessions cleanup (catch-all — this is where session revocation is actually guaranteed, not the best-effort step 2 handler) → collect out_link_ids → Analytics deletion → OutLinks deletion → Profile data + ImageAssets → ConnectedAccounts → AnalysisSessions → QR Code record + SVG asset deletion.

**Cascade idempotency guarantee:** out_link_ids are collected from the database in a read-only step (step 2 of the cascade) before any deletion occurs. Analytics runs before OutLinks are deleted. On retry, if OutLinks are already deleted, the read returns empty and analytics is a safe no-op (already completed).

**Access-blocking guarantee independent of session revocation timing:** every authenticated request additionally verifies `Account.status = 'active'` via a short-TTL cached check (see `08-security-model.md` — "Per-Request Account Status Check") — this bounds how long a deleted account's still-valid access token can be used to a few seconds, regardless of when the Session row itself is revoked.

No "account deletion cancellation" job exists. Deletion is final in V1.
