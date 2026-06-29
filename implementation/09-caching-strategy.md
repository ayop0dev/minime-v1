# Minime V1 — Caching Strategy

## Status

Canonical. Final.

---

## Architecture Authority

```
Architecture-Product-Definition/public-profile/public-profile.cache.policy.v1.md
Architecture-Product-Definition/platform/data/data.retention.policy.v1.md
implementation/04-service-contracts.md
implementation/05-api-contracts.md
```

---

## Purpose

Defines the canonical caching strategy for Minime V1.

Caching is an optimization layer. Caching must never:

- change Profile Content
- change Access decisions
- change Routing decisions
- change Resolution decisions
- become the source of truth

PostgreSQL is the canonical source of truth. Redis stores temporary cache entries only. Cache entries may be discarded at any time. Application correctness must not depend on cache existence.

---

## Cache Architecture

V1 uses two distinct cache layers:

```
Visitor
    │
    ▼
Edge Cache (CDN / Reverse Proxy)   ← public profile HTTP responses
    │
    ▼
Application (NestJS / Redis)        ← internal rendered profile assembly
    │
    ▼
PostgreSQL
```

These two layers are independent. The edge cache operates at HTTP level. The application cache (Redis) operates inside the NestJS API process.

---

## Layer 1 — Edge Cache

**Applies to:** Public profile HTTP responses at `/{username}`

**From `public-profile.cache.policy.v1.md`:**

| Rule | Value |
|---|---|
| Cache TTL | 60 seconds |
| Cache location | CDN, Reverse Proxy, or Edge Network |
| Cache target | `200 OK` responses only |
| `500` responses | Must not be cached |
| `404` responses | May be cached briefly (short TTL) |

**Profile update behavior:**

Immediate cache purge is not required in V1. The platform relies on short TTL expiration (60 seconds) to refresh content naturally. Maximum expected staleness after a profile update: 60 seconds.

**What edge cache does not store:**

- Render Objects
- Themes
- Layouts as independent cache entries

---

## Layer 2 — Application Cache (Redis)

Redis is the only approved distributed cache for application-layer caching.

Application memory must not become a shared cache.

### Approved Cache Entries

| Cache Key | Description | TTL |
|---|---|---|
| `profile:public:{username}` | Rendered public profile data | 60 seconds |
| `profile:content:{account_id}` | Assembled profile content (ProfileContent + Blocks + ConnectedAccounts) | 60 seconds |

These are the only two approved Redis cache keys for profile rendering. There is no `profile:rendered:{profileId}` or `profile:content:{profileId}` — `profileId` does not exist in V1.

### Cache Key Format

```
<domain>:<resource>:<identifier>
```

The identifier must be either `username` (for public resolution) or `account_id` (for ownership-scoped data). There is no `profileId` identifier.

---

## Cache Ownership

Cache ownership follows Product Domain ownership.

- Profile cache is owned by the Rendering domain.
- Each Product Domain owns the cache generated from its own business data.
- Platform Services must not own Product Domain cache policies.
- No Cache Platform Service exists. Redis is infrastructure, not a Platform Service abstraction.

---

## Cache Invalidation

### When to Invalidate

Profile mutations must invalidate both:

```
profile:public:{username}
profile:content:{account_id}
```

Invalidation triggers (from `04-service-contracts.md` and `05-api-contracts.md`):

| Operation | Trigger |
|---|---|
| `updateProfileContent` | Invalidate `profile:public:{username}` + `profile:content:{account_id}` |
| `uploadAvatar` | Invalidate `profile:public:{username}` + `profile:content:{account_id}` |
| `updateAppearance` | Invalidate `profile:public:{username}` + `profile:content:{account_id}` |
| `addBlock` | Invalidate `profile:public:{username}` + `profile:content:{account_id}` |
| `updateBlock` | Invalidate `profile:public:{username}` + `profile:content:{account_id}` |
| `deleteBlock` | Invalidate `profile:public:{username}` + `profile:content:{account_id}` |
| `reorderBlocks` | Invalidate `profile:public:{username}` + `profile:content:{account_id}` |
| `addConnectedAccount` | Invalidate `profile:public:{username}` + `profile:content:{account_id}` |
| `updateConnectedAccount` | Invalidate `profile:public:{username}` + `profile:content:{account_id}` |
| `removeConnectedAccount` | Invalidate `profile:public:{username}` + `profile:content:{account_id}` |
| `reorderConnectedAccounts` | Invalidate `profile:public:{username}` + `profile:content:{account_id}` |
| Account deleted | Invalidate both cache keys for the deleted account |

**Publication changes are not a trigger.** Minime V1 has no publishing workflow.

### Invalidation Timing

Cache invalidation must execute after successful persistence.

Invalidation must not execute before the transaction commits.

Invalidation must not execute before business persistence succeeds.

---

## Cache Population

Cache entries may be populated:

- after successful reads (lazy population on cache miss)
- after successful rendering

Cache population must not modify business entities.

---

## Cache Miss Behavior

Cache misses must execute the canonical business flow:

```
Cache Miss
    │
    ▼
RenderingService reads from repositories (read-only)
    │
    ▼
Populate cache entry
    │
    ▼
Return response
```

Cache misses must not produce errors. Cache misses must not affect business correctness.

---

## Cache Failure Behavior

Redis unavailability must not prevent:

- Database persistence
- Business operation execution
- Response delivery

Cache failures must be logged. Business operations continue without cache on Redis failure.

---

## Cache Expiration

Every cache entry must define expiration. Permanent (no-TTL) cache entries are prohibited.

Cache entries expire automatically. Expired cache entries are regenerated from PostgreSQL on next access.

---

## Cache Security

Cache entries must not store:

- OTP codes
- Access tokens
- Refresh tokens
- Canonical business data as the primary persistence location

Authenticated dashboard responses must not use shared public cache.

---

## Cache Prohibitions

Implementation must not:

- use Redis as the source of truth
- persist business entities in Redis as canonical state
- require cache availability for business correctness
- expose Redis directly to clients or frontend
- bypass PostgreSQL persistence
- create cache entries without TTL
- invalidate cache before successful persistence
- use `profileId` in any cache key — it does not exist
- list "publication changes" as a cache invalidation trigger — no publishing workflow exists
- introduce a Cache Platform Service abstraction over Redis
