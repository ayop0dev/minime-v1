# Social Accounts URL Generation Specification V1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines how Minime generates canonical public profile URLs from normalized social account identifiers.

URL Generation is the final transformation step before the Social Accounts domain produces a Normalized Social Account Handoff Record and delivers it to Connected Accounts for canonical storage.

Its responsibility is intentionally small:

> Combine a normalized identifier with the platform's canonical URL template.

Nothing more.

---

# Design Philosophy

URL Generation is deterministic.

The same platform and the same normalized identifier must always generate exactly the same public URL.

No external dependency may influence the generated result.

---

# Responsibilities

The URL Generation engine is responsible for:

* generating canonical public profile URLs
* applying platform URL templates
* producing deterministic output
* returning the generated URL to the Social Accounts domain

It performs no validation and no communication.

---

# Processing Pipeline

```text
Normalized Identifier

↓

Platform

↓

Platform URL Template

↓

Generated Public URL
```

Generation is a pure transformation.

---

# Input

The URL Generation engine accepts only:

## Platform

Example:

```text
Instagram
```

---

## Normalized Identifier

Example:

```text
hmn
```

Nothing else is required.

---

# Output

The engine returns a canonical public profile URL.

Example:

```text
Platform

Instagram
```

```text
Identifier

hmn
```

↓

```text
https://instagram.com/hmn
```

---

# Canonical URL Templates

The engine never owns URL templates.

Templates are provided by:

```text
Social Platform Rules
```

The URL Generation engine simply applies them.

---

# Deterministic Behavior

Given:

```text
Platform

Instagram
```

and

```text
Identifier

hmn
```

The generated URL must always be:

```text
https://instagram.com/hmn
```

The result must never depend on:

* network access
* APIs
* user session
* cookies
* browser
* platform availability
* server location
* request timing

---

# Stateless Operation

The engine stores no data.

It remembers nothing.

It performs no caching.

Each generation request is completely independent.

---

# Error Handling

The URL Generation engine only reports structural errors.

Examples include:

* unsupported platform
* missing identifier
* missing platform template

It never reports:

* account not found
* invalid account
* unavailable platform
* login required

Those concepts do not exist inside this engine.

---

# No Platform Communication

The engine never:

* opens generated URLs
* performs HTTP requests
* checks DNS
* follows redirects
* resolves short links
* validates destinations

Generation ends immediately after producing the URL string.

---

# Shared Usage

The URL Generation engine is shared by:

* Smart Mode
* Manual Mode

Both modes must always produce identical URLs when provided with the same:

* platform
* normalized identifier

---

# Relationship with Normalization

Normalization executes before URL Generation.

Example:

```text
Input

https://instagram.com/HMn/?utm_source=test
```

↓

```text
Normalization

hmn
```

↓

```text
URL Generation

https://instagram.com/hmn
```

Generation never performs normalization itself.

---

# Relationship with Platform Rules

Platform Rules define:

* URL templates
* formatting conventions

URL Generation consumes those definitions.

It never modifies them.

---

# Design Principles

## Pure Transformation

URL Generation performs only one operation:

Identifier → URL

---

## Deterministic

Identical input always produces identical output.

---

## Stateless

No persistence.

No cache.

No session.

---

## Platform Independent

Adding a new platform requires only a new URL template.

The generation engine itself never changes.

---

## Zero External Dependencies

The engine operates entirely offline.

Internet connectivity must never affect generated URLs.

---

# Canonical Statement

The URL Generation engine is a pure, deterministic transformation component.

It combines a normalized social account identifier with a platform-defined URL template to produce a canonical public profile URL.

Its behavior is fully described by this single transformation. Nothing beyond it occurs.
