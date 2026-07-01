# SEO Indexing Policy V1

## Status

Approved.

**Governing decision:** This policy previously described a draft/published visibility workflow that does not exist anywhere in Minime V1. Minime V1 has no publishing workflow (`implementation/README.md` — "No Publishing Workflow"; `appearance.system.specification.v1.md`; `profile.content.specification.v1.md`). This document has been corrected to key indexing eligibility exclusively off `Account.status`, consistent with every other V1 domain. See `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` — APD-013.

---

## Purpose

This policy defines when public Minime pages may participate in search engine indexing.

The objective is to ensure that only appropriate public content becomes discoverable by search engines while protecting suspended and deleted accounts' data.

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

The SEO capability does not determine whether a profile is public.

Public accessibility is owned by the Account domain, expressed entirely through `Account.status`.

SEO consumes that value only. There is no separate "publication status" or "visibility" domain concept in V1 — see `account/account.model.specification.v1.md`.

---

# Indexing Eligibility

A page may participate in search indexing only if:

* the account resolves to an existing `Account` record, and
* `Account.status = 'active'`.

That is the entire eligibility rule in V1. SEO must never override this or introduce any additional condition.

---

# Non-Indexable Pages

The following must never be indexed:

* accounts with `Account.status = 'suspended'`
* accounts with `Account.status = 'deleted'`
* usernames that do not resolve to any account
* error pages
* temporary system pages

There is no "draft," "unpublished," or "private" profile state in V1 to exclude — every active account's public profile is, by definition, public and indexable. A profile exists and is public as soon as the owning account reaches `status = 'active'` (see `account/account.model.specification.v1.md` and the Account Claim registration flow); there is no intermediate state between account creation and public visibility.

Additional non-public pages (e.g. future internal or administrative surfaces) may be excluded by future platform policies.

---

# Robots Policy

Robots directives must always reflect `Account.status`:

* `Account.status = 'active'` → `index, follow`
* `Account.status = 'suspended'` or `'deleted'` → `noindex, nofollow` (the page itself returns `404`, per `public-profile.error.states.v1.md`; the robots directive is a defense-in-depth statement for any cached or previously-crawled representation)

SEO automatically generates the correct robots directive from `Account.status`. Users are not expected to configure robots manually, and no user-facing control to change indexing exists in V1.

---

# Canonical Policy

Every indexable page must expose one canonical URL.

Canonical URLs must be:

* unique
* stable
* publicly accessible
* deterministic

The canonical URL is always `{APP_BASE_URL}/{Account.username}`. Because usernames are immutable in V1, canonical URLs never change for the lifetime of an active account.

---

# Sitemap Participation

Only accounts with `Account.status = 'active'` may participate in generated sitemaps.

Suspended and deleted accounts must be excluded from sitemaps.

Sitemap generation must remain fully automatic.

---

# Duplicate Content

The platform should minimize duplicate public URLs whenever possible.

Each active account has exactly one canonical public identity: `/{username}`. There is no alternate route to the same profile in V1 (the QR redirect at `/qr/{qr_code_id}` is a redirect to `/{username}`, not an independently indexable representation — see `qr-code/qr-code.system.specification.v1.md`).

---

# Account Status Changes

Whenever `Account.status` changes, indexing behavior must automatically follow within the normal cache TTL (≤ 60 seconds, per `public-profile.cache.policy.v1.md`):

* `active → suspended`: page returns `404`; robots becomes `noindex, nofollow`
* `active → deleted`: page returns `404`; robots becomes `noindex, nofollow`
* `suspended → active` (administrative restoration): page resumes serving; robots returns to `index, follow`

There is no `Published → Draft` or `Draft → Published` transition in V1; no such states exist. No manual synchronization is ever required — indexing behavior is a pure function of `Account.status` at request time.

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

The evaluation process should avoid unnecessary runtime complexity. Determining eligibility requires reading a single field (`Account.status`) already loaded during normal profile resolution — no additional query is required.

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
* any concept of draft, unpublished, scheduled, preview, or private profile state (these do not exist in V1 — see `implementation/README.md`, "No Publishing Workflow")

The platform should expose a single, predictable indexing policy.

---

# Relationship With Other Domains

## Account

`Account.status` is the single source of truth for indexing eligibility. SEO reads it; SEO never writes to it and never introduces a parallel visibility concept.

---

## Public Profile

Determines whether a profile request returns `200` or `404`, using the same `Account.status` check. SEO's robots/indexing decision and Public Profile's serve/404 decision are always consistent because both derive from the same field.

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

* Only accounts with `Account.status = 'active'` participate in search indexing.
* Private/suspended/deleted account information is never exposed through indexing.
* Canonical URLs remain unique and stable.
* Sitemap participation follows `Account.status` automatically.
* Indexing behavior remains deterministic across the platform.
* Users never need to configure technical indexing settings.
* No document describes a draft/publish workflow that does not exist in V1.
