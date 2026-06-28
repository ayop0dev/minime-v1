# SEO Architecture Specification V1

## Purpose

This document defines the architecture, ownership boundaries, responsibilities, and operating principles of the SEO Domain.

The SEO Domain exists to describe public Minime pages for search engines and social platforms.

It never owns business data.

It never changes business data.

It only transforms existing public information into standardized SEO output.

---

# Domain Owner

The SEO Domain owns how public pages are described to search engines, crawlers, and social platforms.

---

# Architectural Role

Within Minime V1, SEO acts as a read-only domain.

It consumes public information from other domains and produces SEO artifacts that are rendered into public pages.

SEO never becomes the source of truth for any business information.

---

# Architectural Classification

The SEO Domain is a **Cross-Cutting Product Capability**.

It is not a Product Domain.

It is not a Platform Service.

## Why Not a Product Domain

The SEO Domain owns no product journey stage.

The SEO Domain owns no canonical entity.

The SEO Domain has no user-facing lifecycle.

The SEO Domain does not appear in the Domain Coverage table.

Product Domains own business behavior. SEO generates descriptions of that behavior for external systems.

## Why Not a Platform Service

The SEO Domain is not infrastructure.

The SEO Domain reads product-level data (Profile Content, Public Profile, Account, Appearance).

The SEO Domain is not one of the four Platform Services: Data, Storage, Events, AI.

Platform Services own shared technical capabilities. SEO is a product-facing read-only capability.

## Cross-Cutting Scope

SEO applies to every public profile without being owned by any single domain.

It reads from Profile Content, Public Profile, Account, and Appearance without owning any of them.

No single Product Domain is responsible for SEO. SEO is responsible to all of them equally.

## Boundary Summary

| Dimension | Value |
| --- | --- |
| Classification | Cross-Cutting Product Capability |
| Owner | SEO Domain |
| Owned Entities | None |
| Writes To | No domain |
| Reads From | Account, Profile Content, Public Profile, Appearance |
| Produces | Metadata definitions (HTML title, canonical URL, robots, OG, Twitter Card, Structured Data) |
| Consumed By | Rendering |
| Dependency Direction | SEO → Rendering (Rendering consumes SEO definitions; SEO does not depend on Rendering) |

## Non-Goals

The SEO Domain does not:

* own any canonical entity
* write to any domain
* define rendering behavior
* replace any Product Domain
* provide infrastructure capability
* directly emit HTML

---

# Domain Responsibilities

The SEO Domain owns:

* metadata generation
* canonical URL generation
* robots directives
* Open Graph metadata
* Twitter Card metadata
* structured data generation
* social preview metadata
* sitemap participation
* search indexing policies
* SEO rendering requirements

---

# Domain Boundaries

The SEO Domain may read:

* Profile Content
* Public Profile
* Account
* Appearance
* Publishing Status

The SEO Domain never writes to those domains.

---

# Source of Truth

SEO never creates business information.

Every SEO artifact must be derived from canonical data owned elsewhere.

Examples:

* Display Name
* Handle
* Bio
* Avatar
* Cover Image
* Profile URL

remain owned by their respective domains.

---

# Generated Assets

The SEO Domain generates:

* HTML title
* Meta description
* Canonical URL
* Robots directives
* Open Graph metadata
* Twitter Card metadata
* Structured Data
* Social Preview metadata

These artifacts are generated at render time or during publishing depending on implementation.

Implementation details are intentionally outside the scope of this specification.

---

# Domain Dependencies

SEO depends on:

* Account
* Profile Content
* Public Profile
* Appearance
* Rendering

SEO has no dependency on:

* Analytics
* Integrations
* AI
* Events
* Storage implementation

---

# Rendering Contract

Rendering is responsible for producing HTML.

SEO is responsible for defining which SEO artifacts must exist.

Rendering must never invent SEO information.

SEO must never render HTML directly.

---

# Automatic Generation

SEO artifacts should be generated automatically whenever possible.

Manual configuration should be avoided.

The platform should derive SEO information from existing public profile information.

---

# Deterministic Output

The same public profile should always generate the same SEO output unless the underlying profile data changes.

SEO generation must be deterministic.

No randomness is permitted.

---

# Performance Principles

SEO generation should remain:

* lightweight
* cache-friendly
* deterministic
* implementation-independent

SEO must never introduce expensive runtime processing.

---

# Security Principles

SEO must never execute user-provided code.

SEO must never expose private information.

SEO must never generate metadata from non-public data.

SEO must never bypass profile visibility rules.

---

# Privacy Rules

Private profiles must never expose SEO metadata that reveals hidden information.

Only publicly available information may be transformed into SEO artifacts.

---

# V1 Constraints

The SEO Domain intentionally excludes:

* SEO dashboards
* keyword optimization
* ranking analysis
* backlink management
* search console integrations
* AI-generated SEO optimization
* manual metadata editing
* custom meta tags
* custom HTML
* custom JavaScript

---

# Success Criteria

The architecture is considered successful when:

* SEO remains completely independent from business domains.
* Every public profile produces valid SEO metadata.
* Rendering receives a complete SEO definition.
* Search engines can correctly understand every published profile.
* Users do not need SEO knowledge to publish successfully.
* The domain remains lightweight, deterministic, and easy to maintain.
