# Public Profile Cache Policy V1

## Status

Approved

**Governing decision:** This policy previously prohibited "Smart Cache Invalidation" and stated that profile updates rely on TTL expiration alone. The approved implementation (`implementation/09-caching-strategy.md`) requires immediate, mutation-triggered invalidation of the application-level cache after every profile-mutating operation, which is a stronger freshness guarantee than TTL-only expiration. This policy has been updated to adopt that behavior as the official V1 cache strategy. See `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` — APD-014.

---

# Purpose

This document defines caching behavior for public profile delivery in Minime V1.

The goal is to improve response speed and reduce infrastructure load while keeping profile updates reasonably fresh.

---

# Cache Strategy

V1 uses two independent cache layers:

```text
Layer 1 — Edge Cache (CDN / Reverse Proxy)   — public profile HTTP responses
Layer 2 — Application Cache (Redis)          — assembled profile content, invalidated on mutation
```

Layer 2 is invalidated immediately (Smart Cache Invalidation) after every successful profile-mutating write. Layer 1 relies on TTL expiration only, bounded by the `Cache-Control: max-age` value computed from Layer 2's remaining TTL (see "Freshness Expectations" below) — the edge layer never receives an explicit purge call in V1.

The platform does not use:

```text
No Cache

Full Profile Cache (a cache with no TTL and no invalidation)

Static Profile Generation
```

---

# Cache Philosophy

Caching is an optimization layer.

Caching must never change:

```text
Profile Content

Access Decisions

Routing Decisions

Resolution Decisions
```

Caching only improves delivery speed.

---

# Cache Scope

Cache applies to:

```text
Public Profile Responses
```

Examples:

```text
https://minime.ae/ahmed

https://minime.ae/hala

https://minime.ae/omar
```

---

# Cache Location

V1 cache should exist at the edge layer.

Examples:

```text
CDN

Reverse Proxy

Edge Network
```

Implementation is platform-specific.

---

# Cache TTL

Official V1 TTL:

```text
60 Seconds
```

After expiration:

```text
Request
↓
Revalidate
↓
Generate Fresh Response
↓
Cache Again
```

---

# Profile Updates

When profile content changes, the application cache (Layer 2 — Redis) is invalidated immediately, synchronously, as part of the same request that performed the mutation, after persistence succeeds:

```text
Profile Mutation Persisted
↓
Invalidate profile:public:{username} and profile:content:{account_id} (Redis)
↓
Next Read Recomputes From PostgreSQL
```

This is Smart Cache Invalidation: invalidation is triggered by the specific mutation, not solely by elapsed time. The full list of mutating operations that trigger invalidation is defined in `implementation/09-caching-strategy.md` — "Cache Invalidation" (covers `updateProfileContent`, `uploadAvatar`, `updateAppearance`, `addBlock`, `updateBlock`, `deleteBlock`, `reorderBlocks`, all `ConnectedAccount` mutations, and `gtm_container_id` changes).

The edge cache (Layer 1) is **not** explicitly purged on mutation in V1 — it continues to rely on TTL expiration, bounded by the shared freshness budget described below. Explicit edge purge (a CDN-level API call) remains out of V1 scope; only the application-level Redis cache is actively invalidated.

Cache invalidation failure (e.g. Redis unavailable) must never block or fail the mutation itself — the write to PostgreSQL is the operation of record, and invalidation is a best-effort follow-up step. See `implementation/09-caching-strategy.md` — "Cache Failure Behavior."

---

# Freshness Expectations

Maximum expected delay after profile updates:

```text
60 Seconds
```

This is a single end-to-end budget across every cache layer between PostgreSQL and the visitor, not 60 seconds granted independently to each layer. Where more than one cache layer exists (e.g. an application-level cache in front of PostgreSQL, and this document's edge/CDN layer in front of that), the total combined staleness must never exceed 60 seconds. See `implementation/09-caching-strategy.md` — "Freshness Budget Propagation" for the mechanism.

or less.

---

# Cacheable Responses

Cache:

```text
200 OK
```

responses.

---

# Non-Cacheable Responses

Do not cache:

```text
500 Internal Server Error
```

responses.

---

# 404 Responses

404 responses may be cached briefly.

Recommended:

```text
Short TTL
```

to avoid unnecessary load from repeated invalid requests.

---

# Relationship With Routing

Routing always executes before cache evaluation logic.

Caching never replaces routing.

---

# Relationship With Resolution

Resolution remains the source of truth.

Caching must never permanently store Account ownership decisions.

---

# Relationship With Access Policy

Access Policy decisions remain the source of truth.

Content changes are reflected in cached responses immediately for Layer 2 (Smart Cache Invalidation) and within the shared freshness budget for Layer 1 (TTL expiration, bounded to the remaining Layer 2 TTL at the moment of each response — see `implementation/09-caching-strategy.md` — "Freshness Budget Propagation").

---

# Relationship With Presentation

Caching stores delivered profile responses.

Caching does not store:

```text
Render Objects

Themes

Layouts
```

as independent cache entities in V1.

---

# V1 Scope

Supported:

```text
Edge Cache (Layer 1), TTL-bound
Application Cache (Layer 2, Redis), TTL-bound
Smart Cache Invalidation of Layer 2 on profile mutation
Shared freshness budget across both layers (see implementation/09-caching-strategy.md)
Public Profile Caching
404 Caching
```

Not Supported:

```text
Full Profile Cache (cache with no TTL)

Static Generation

Manual/explicit Edge (CDN) Purge — Layer 1 relies on TTL only, never an explicit purge call

Cache Versioning
```

---

# Architectural Rule

Caching must remain optional.

The platform must continue functioning correctly even if caching is disabled.

---

# Core Principle

Caching improves speed.

Caching does not change behavior.

The system remains correct with or without cache.
