# SEO Indexing Policy V1

## Purpose

This policy defines when public Minime pages may participate in search engine indexing.

The objective is to ensure that only appropriate public content becomes discoverable by search engines while protecting unpublished and private information.

This policy defines indexing behavior only.

It does not define rendering, metadata generation, or page ownership.

---

# Policy Principles

Indexing decisions must always be:

* automatic
* deterministic
* privacy-first
* implementation-independent
* derived from canonical data

Users should never need advanced SEO knowledge to publish safely.

---

# Source of Truth

The SEO Domain does not determine whether a profile is public.

Publication status and visibility are owned by their respective domains.

SEO consumes those decisions.

---

# Indexing Eligibility

A page may participate in search indexing only if all required publication conditions are satisfied.

Examples include:

* the profile is published
* the profile is publicly accessible
* the profile is not disabled
* the profile is not restricted

SEO must never override business visibility rules.

---

# Non-Indexable Pages

The following pages must never be indexed:

* draft profiles
* unpublished profiles
* suspended profiles
* deleted profiles
* private profiles
* restricted profiles
* error pages
* temporary system pages

Additional non-public pages may be excluded by future platform policies.

---

# Robots Policy

Robots directives must always reflect the current visibility state.

The generated robots metadata must never contradict business visibility.

SEO automatically generates the correct robots directives.

Users are not expected to configure robots manually.

---

# Canonical Policy

Every indexable page must expose one canonical URL.

Canonical URLs must be:

* unique
* stable
* publicly accessible
* deterministic

Canonical URLs must never reference private or temporary locations.

---

# Sitemap Participation

Only indexable public pages may participate in generated sitemaps.

Pages that are excluded from indexing must also be excluded from sitemaps.

Sitemap generation must remain fully automatic.

---

# Duplicate Content

The platform should minimize duplicate public URLs whenever possible.

Each public profile should have one canonical public identity.

SEO should avoid exposing multiple indexable representations of the same profile.

---

# Visibility Changes

Whenever profile visibility changes, indexing behavior must automatically follow.

Examples include:

Public → Private

Private → Public

Published → Draft

Draft → Published

No manual synchronization should ever be required.

---

# Public Data Protection

SEO must never expose information that is unavailable through the public profile itself.

Changing indexing behavior must never reveal hidden profile information.

---

# Search Engine Independence

The policy defines the intended indexing behavior.

Search engines remain responsible for their own crawling and indexing decisions.

The platform cannot guarantee indexing.

---

# Performance

Indexing rules should remain:

* lightweight
* deterministic
* inexpensive
* cache-friendly

The evaluation process should avoid unnecessary runtime complexity.

---

# V1 Constraints

V1 intentionally excludes:

* per-page indexing overrides
* custom robots rules
* custom canonical rules
* search engine specific configuration
* manual sitemap management
* crawl budget controls
* indexing analytics
* search engine integrations

The platform should expose a single, predictable indexing policy.

---

# Relationship With Other Domains

## Public Profile

Determines whether a profile is publicly available.

SEO respects this decision.

---

## Rendering

Outputs robots metadata, canonical URLs, and sitemap references defined by this policy.

---

## Metadata

Generates metadata only for pages permitted by this policy.

---

## Integrations

Integrations have no influence on indexing decisions.

---

## Analytics

Analytics has no influence on indexing decisions.

---

# Success Criteria

This policy is considered successful when:

* Only eligible public profiles participate in search indexing.
* Private information is never exposed through indexing.
* Canonical URLs remain unique and stable.
* Sitemap participation follows visibility automatically.
* Indexing behavior remains deterministic across the platform.
* Users never need to configure technical indexing settings.
