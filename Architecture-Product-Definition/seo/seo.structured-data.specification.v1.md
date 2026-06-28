# SEO Structured Data Specification V1

## Purpose

This specification defines how structured data is generated for public Minime pages.

The goal is to help search engines understand the meaning of a public profile through standardized structured data without requiring user configuration.

Structured data is generated automatically from canonical public information.

---

# Principles

Structured data must always be:

* automatic
* deterministic
* standards-based
* implementation-independent
* derived from canonical data

Users should never be required to manually configure structured data.

---

# Ownership

The SEO Domain owns structured data generation.

It does not own the profile information used to generate it.

All structured data is derived from public canonical data.

---

# Source of Truth

Structured data consumes information from:

* Account
* Profile Content
* Public Profile
* Appearance

These domains remain the source of truth.

---

# Supported Schema Types

V1 intentionally supports only a small number of schema types.

The platform should avoid unnecessary complexity.

Supported schema types include:

* ProfilePage
* Person
* Organization
* WebSite

The generated schema depends on the type of public profile.

---

# Profile Type Resolution

The platform determines the most appropriate schema automatically.

Users should never choose schema types manually.

Future versions may introduce additional profile classifications if necessary.

---

# Automatic Generation

Structured data must be generated automatically whenever a public profile is rendered or published.

No manual synchronization is required.

---

# Canonical Consistency

Structured data must always describe the same public entity represented by:

* HTML metadata
* Open Graph metadata
* Twitter metadata
* Canonical URL

All generated SEO artifacts must remain consistent.

---

# Public Data Only

Structured data must never expose:

* private information
* hidden fields
* unpublished content
* draft information
* internal identifiers
* implementation details

Only publicly available information may appear.

---

# Rendering Contract

The SEO Domain defines structured data.

The Rendering Domain serializes it into the final HTML document.

Rendering must not generate schema independently.

---

# Performance

Structured data generation should remain lightweight.

The generation process should support efficient caching and deterministic output.

---

# Validation

Generated structured data should:

* follow Schema.org standards
* remain syntactically valid
* remain internally consistent
* avoid duplicate entities
* avoid conflicting definitions

---

# Extensibility

Future schema types may be added without changing the ownership model.

New schema definitions should extend the existing generation pipeline rather than introducing separate systems.

---

# V1 Constraints

V1 intentionally excludes:

* manual schema editing
* custom JSON-LD
* custom schema injection
* organization verification
* business-specific schema
* product schema
* article schema
* event schema
* FAQ schema
* AI-generated schema

The platform should support only the schema necessary to accurately describe a public Minime profile.

---

# Success Criteria

This specification is considered successful when:

* Every public profile exposes valid structured data.
* Structured data is generated automatically.
* Search engines can correctly identify the public profile.
* Structured data remains consistent with all other SEO metadata.
* Users never need to understand Schema.org to publish a technically correct profile.
