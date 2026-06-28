# SEO Metadata Specification V1

## Purpose

This specification defines how SEO metadata is generated for every public Minime page.

The objective is to guarantee that every published profile exposes complete, consistent, and technically correct metadata without requiring manual SEO configuration.

---

# Principles

Metadata generation must always be:

* automatic
* deterministic
* implementation-independent
* cache-friendly
* derived from canonical data

SEO metadata must never become the source of truth.

---

# Ownership

The SEO Domain owns metadata generation.

It does not own the underlying profile information.

Metadata is always derived from canonical data owned by other domains.

---

# Metadata Lifecycle

The lifecycle is:

```text
Canonical Data
        │
        ▼
SEO Metadata Generation
        │
        ▼
Rendering
        │
        ▼
Public HTML
```

SEO metadata exists only as a generated representation of public information.

---

# Metadata Sources

Metadata may consume information from:

* Account
* Profile Content
* Appearance
* Public Profile

No additional user input is required.

---

# Required Metadata

Every public profile must expose:

## HTML Title

Represents the primary page title.

Generated automatically.

---

## Meta Description

Summarizes the public profile.

Generated automatically from public profile information.

---

## Canonical URL

Every public page must expose exactly one canonical URL.

Canonical URLs are always generated automatically.

---

## Robots

Defines whether search engines may index the page.

Robots directives must reflect the publication state and visibility rules.

---

## Open Graph

Every public profile must expose complete Open Graph metadata for social sharing.

Open Graph metadata should include:

* title
* description
* URL
* image
* content type

---

## Twitter Card

Twitter metadata should mirror Open Graph whenever possible.

The platform should avoid maintaining two independent metadata sources.

---

## Social Preview Image

A preview image should always be available whenever possible.

The image should be generated from existing public assets.

SEO never owns those assets.

---

## Language

Metadata language should follow the rendered page language.

---

# Automatic Generation Rules

Whenever profile information changes, metadata should automatically reflect the latest public state.

No manual synchronization should ever be required.

---

# Fallback Strategy

Metadata generation must remain resilient.

If optional information is unavailable, the platform should gracefully fall back to other public information.

Fallbacks must always remain deterministic.

The platform must never generate placeholder or misleading metadata.

---

# Consistency Rules

All metadata generated for a page must describe the same public profile.

Different metadata formats must never contradict each other.

Examples include:

* HTML Title
* Open Graph Title
* Twitter Title

These should represent the same public identity.

---

# Visibility Rules

Only public information may appear in metadata.

Hidden, draft, private, or restricted information must never be exposed through SEO metadata.

---

# Rendering Contract

Rendering is responsible for emitting metadata into the final HTML document.

The SEO Domain is responsible for defining what metadata exists.

The Rendering Domain must not invent metadata independently.

---

# Performance

Metadata generation should remain inexpensive.

The generation process should avoid unnecessary computation and support efficient caching.

---

# Security

Metadata must never include:

* executable code
* user-provided HTML
* JavaScript
* hidden identifiers
* internal references
* private information

All generated values must be safe for public output.

---

# V1 Scope

V1 intentionally supports automatic metadata generation only.

The following capabilities are outside the scope of V1:

* manual SEO title
* manual meta description
* custom canonical URLs
* custom robots directives
* custom Open Graph fields
* custom Twitter metadata
* per-page SEO customization
* AI-generated metadata optimization

---

# Success Criteria

This specification is considered successful when:

* Every published profile exposes complete metadata.
* Metadata is generated automatically.
* Metadata remains consistent across all formats.
* Metadata never exposes private information.
* Rendering receives a complete metadata definition.
* Users never need to configure technical SEO settings.
