# Minime V1 — Event Contracts

## Status

Reconciled. Supersedes `07-event-contracts.md`.

---

## Architecture Authority

```
Architecture-Product-Definition/docs/MINIME_V1_PRODUCT_ARCHITECTURE_MAP.md
implementation/04-service-contracts.md
```

---

## Purpose

Defines the canonical internal event contracts for Minime V1.

Event contracts define:

- event ownership
- event naming
- event payload structure
- delivery guarantees
- handler rules

---

## Technology: NestJS EventEmitter

Minime V1 uses **NestJS EventEmitter** as the internal event bus.

**Actual delivery characteristics:**

- Events are dispatched **in-process**, within the same Node.js instance.
- Delivery is **synchronous** (or async within the same process using `emitAsync`).
- Events are **not durable**. An unhandled exception or process restart means the event is lost.
- There is **no automatic retry** for failed handlers.
- There is **no distributed delivery**. Events do not cross process boundaries.
- There is **no queue**. EventEmitter has no backpressure or persistence.

**Consequence for retry:**

If a handler needs retry-on-failure behavior, the handler must enqueue a BullMQ job. The BullMQ job owns the retry logic, not EventEmitter.

**Example pattern:**

```
Service publishes event via EventEmitter
    ↓
EventEmitter handler executes in-process
    ↓
Handler enqueues BullMQ job if durable work is needed
    ↓
BullMQ worker executes with retry policy
```

Claims of distributed event delivery, cross-service event routing, or automatic retry via EventEmitter are incorrect and must not appear in implementation code or comments.

---

## Event Publication Rules

- Events must be published only after successful database persistence.
- Events must represent completed business operations.
- Events must be immutable after publication.
- Event publication must not modify business state.
- Failed business operations must not publish success events.
- Handler failures must not roll back completed business persistence.
- Handler failures must be logged.

---

## Event Naming

Format: `domain.action` or `domain.entity.action`

```text
account.created
auth.otp.requested
profile.block.added
connected-account.removed
out-link.clicked
```

Rules:
- Lowercase
- Dot-separated segments
- Names must remain stable after deployment

---

## Event Payload Rules

Every event payload must contain:

```
event_id     — UUID, unique per event emission
occurred_at  — UTC timestamp of the business operation
account_id   — owning account (where applicable)
entity_id    — primary identifier of the affected entity
```

Additional domain-specific fields may be included when required by subscribers.

Payloads must not contain:
- Passwords or OTP codes
- Access tokens or refresh tokens
- Authentication secrets
- Sensitive personal information beyond what subscribers require

---

## Event Handler Rules

- Handlers must be independent.
- Handlers must not assume execution order.
- Handlers must not mutate unrelated Product Domains.
- Handlers must be written defensively (handle errors without crashing the process).
- Handlers that need retry must enqueue a BullMQ job, not rely on EventEmitter re-emission.

---

## Canonical Business Event Catalog

### Account Domain

| Event | Publisher | Trigger |
|---|---|---|
| `account.created` | AccountService | Registration completed; Account created |
| `account.settings.updated` | AccountService | Account.settings mutated |
| `account.qr.updated` | AccountService | Account.qr_config mutated |
| `account.deleted` | AccountService | Account.status set to 'deleted' |

**Removed:** `account.deactivated`, `account.reactivated`, `account.deletion.requested`, `account.deletion.cancelled` — these operations do not exist in V1.

---

### Authentication Domain

| Event | Publisher | Trigger |
|---|---|---|
| `auth.otp.requested` | AuthService | OtpVerification record created |
| `auth.otp.verified` | AuthService | OTP verified; OtpVerification deleted |
| `auth.google.authenticated` | AuthService | Google ID token validated |
| `auth.registration.completed` | AuthService | Account + AuthCredential + ProfileContent created |
| `auth.session.created` | AuthService | Session record created |
| `auth.session.refreshed` | AuthService | Refresh token rotated |
| `auth.session.revoked` | AuthService | Session.revoked_at set |
| `auth.logout` | AuthService | Current session revoked via logout |

---

### Username Domain

| Event | Publisher | Trigger |
|---|---|---|
| `username.reserved` | UsernameService | UsernameReservation created |
| `username.reservation.expired` | UsernameService (via BullMQ job) | Reservation hard-deleted after expiry |

**Removed:** `username.changed`, `username.assigned`, `username.released` — these operations do not exist in V1. Username is immutable after registration.

---

### Profile Domain

| Event | Publisher | Trigger |
|---|---|---|
| `profile.updated` | ProfileService | ProfileContent fields mutated |
| `profile.avatar.updated` | ProfileService | avatar_storage_key updated |
| `profile.appearance.updated` | ProfileService | ProfileContent.appearance_config mutated |
| `profile.image.uploaded` | ProfileService | ImageAsset record created |
| `profile.image.deleted` | ProfileService | ImageAsset record hard-deleted |
| `profile.block.added` | ProfileService | Block record created |
| `profile.block.updated` | ProfileService | Block.content or Block.settings mutated |
| `profile.block.deleted` | ProfileService | Block.deleted_at set |
| `profile.block.reordered` | ProfileService | Block.sort_order values updated |

**Removed:** `profile.published`, `profile.unpublished` — there is no publishing workflow in V1. Saving a change makes it visible within ≤ 60 seconds.

**Removed:** `profile.created` — ProfileContent is created as part of registration; the creation is covered by `auth.registration.completed`.

---

### Social Accounts Domain

**No events.** Social Accounts is a processing domain. It owns no persistent state and emits no business events.

**Removed:** `social-account.added`, `social-account.updated`, `social-account.removed`, `social-account.verified`, `social-account.reordered` — Social Accounts domain has no persistent entities and no business lifecycle events.

---

### Connected Accounts Domain

| Event | Publisher | Trigger |
|---|---|---|
| `connected-account.added` | ConnectedAccountsService | ConnectedAccount created |
| `connected-account.updated` | ConnectedAccountsService | ConnectedAccount.username (and regenerated url) updated |
| `connected-account.removed` | ConnectedAccountsService | ConnectedAccount hard-deleted |
| `connected-account.reordered` | ConnectedAccountsService | sort_order values updated |

**Removed:** `provider.connected`, `provider.disconnected`, `provider.connection.refreshed`, `provider.connection.revoked` — these refer to OAuth provider connections which do not exist in V1. ConnectedAccount is a lightweight social link record, not an OAuth connection.

---

### Out Links Domain

| Event | Publisher | Trigger |
|---|---|---|
| `out-link.created` | OutLinksService | OutLink created with status 'active' |
| `out-link.archived` | OutLinksService | OutLink.status set to 'archived' |
| `out-link.clicked` | OutLinksService (via BullMQ job) | Public redirect resolved; click recorded asynchronously |

**Removed:** `out-link.deleted`, `out-link.enabled`, `out-link.disabled` — these operations do not exist in V1. There is no is_enabled field. Lifecycle uses status: active → archived → purge.

**Note on `out-link.clicked`:** This event is published by a BullMQ job, not directly by EventEmitter, because click recording must not block the redirect response. The BullMQ job calls AnalyticsService.recordLinkClick and then emits `out-link.clicked`.

---

### Analytics Domain

| Event | Publisher | Trigger |
|---|---|---|
| `analytics.retention.completed` | AnalyticsService (via BullMQ job) | Retention cleanup job completed |

**Note:** Analytics event recording (`recordProfileView`, `recordLinkClick`) is a command, not an event. AnalyticsService does not publish an event for every recorded measurement.

---

## Event Consumers (Handlers)

The following handlers subscribe to events from other domains:

| Handler | Subscribes to | Action |
|---|---|---|
| RenderingService cache invalidator | `profile.updated`, `profile.avatar.updated`, `profile.appearance.updated`, `profile.block.*` | Invalidates `profile:public:{username}` and `profile:content:{account_id}` in Redis |
| Session revocation handler | `account.deleted` | Calls AuthService.revokeAllSessions for the deleted account (synchronous, in-process — satisfies the "at the moment of deletion" security requirement) |
| Deletion cascade enqueuer | `account.deleted` | Enqueues `account.deletion.cascade` BullMQ job for durable cross-domain cascade cleanup |
| Analytics ingestion (BullMQ) | `out-link.clicked` payload via job | Calls AnalyticsService.recordLinkClick |

---

## Event Flow

Every business event follows this flow:

```
Business Operation Executes
    ↓
Repository Persistence Succeeds
    ↓
Transaction Commits
    ↓
EventEmitter publishes event (in-process)
    ↓
Handlers execute (synchronous within same process)
    ├──► Cache invalidation (synchronous)
    └──► BullMQ job enqueue (if durable async work needed)
              ↓
         BullMQ worker executes (with retry)
```

Event publication before persistence commit is prohibited.

---

## Event Prohibitions

Implementation must not:

- publish events before database persistence succeeds
- mutate business entities inside unrelated event handlers
- depend on handler execution order
- use events as the source of truth for business state
- store business state inside event payloads
- expose secrets or tokens through event payloads
- claim EventEmitter provides distributed delivery, queue-based retry, or cross-process guarantees
- publish `profile.published` or `profile.unpublished` — these concepts do not exist
- publish `username.changed` — username changes are not supported in V1
- publish `provider.*` events — OAuth provider connections are not in V1
- publish `social-account.*` events — Social Accounts domain emits no events
- publish `out-link.enabled` or `out-link.disabled` — no is_enabled field exists
- publish `qr_scan` analytics events — qr_scan is not a V1 analytics event type
