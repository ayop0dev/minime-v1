# Public Profile Cache Policy V1

## Status

Approved

---

# Purpose

This document defines caching behavior for public profile delivery in Minime V1.

The goal is to improve response speed and reduce infrastructure load while keeping profile updates reasonably fresh.

---

# Cache Strategy

V1 uses:

```text
Basic Edge Cache
```

The platform does not use:

```text
No Cache

Full Profile Cache

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

When profile content changes:

```text
Immediate Cache Purge
```

is not required in V1.

The platform relies on:

```text
Short TTL Expiration
```

to refresh content naturally.

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

Content changes are reflected in cached responses after TTL expiration.

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
Basic Edge Cache

Short TTL

Public Profile Caching

404 Caching
```

Not Supported:

```text
Full Profile Cache

Static Generation

Manual Cache Purge

Smart Cache Invalidation

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
