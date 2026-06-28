# Minime Data Retention Policy V1

**Status:** Canonical
**Version:** V1
**Platform Service:** Data
**Parent:** `platform/data/data.architecture.specification.v1.md`
**Repository State:** Canonical Repository State V1.0

---

# 1. Purpose

This document defines the retention policy for structured data in Minime V1.

Retention answers one question:

> How long should information remain part of Minime?

Retention is not only about deletion.

Retention defines the complete long-term treatment of data.

Possible outcomes include:

* keep
* archive
* anonymize
* regenerate
* expire
* delete

Every meaningful category of data should have an intentional retention strategy.

---

# 2. Core Principle

Minime stores information because it serves the product.

Minime does not store information simply because it can.

Every retained record should have a reason to exist.

If no reason exists, the record should eventually disappear.

---

# 3. Retention Principles

Every retention decision should follow these principles.

---

## Keep Only What Matters

Information should remain only while it provides value to:

* the user
* the product
* required system operations
* legal or operational obligations

---

## Minimize Storage

Smaller systems are:

* cheaper
* faster
* simpler
* easier to maintain

Minime prefers removing unnecessary information instead of keeping it forever.

---

## User Data Is Valuable

User-approved canonical data is the highest-value data category.

It should not disappear unexpectedly.

Removing user-owned canonical data requires explicit lifecycle rules.

---

## Temporary Means Temporary

Temporary information should never become permanent accidentally.

Every temporary record should eventually:

* expire
* promote
* or disappear.

---

## Generated Data Is Replaceable

Generated data should usually be regenerated instead of retained forever.

If regeneration is inexpensive, long-term storage should be questioned.

---

## Derived Data Is Disposable

Derived data exists for convenience.

Canonical data always has higher value.

If Derived Data is lost, the system should prefer recalculation whenever practical.

---

# 4. Retention Outcomes

Every data category should eventually reach one of these outcomes.

---

## Keep

The record remains active.

Example:

* Account
* Profile Content
* Blocks

---

## Archive

The record is no longer active but still has historical value.

Archived records should not participate in normal product behavior.

---

## Expire

The record disappears automatically after serving its purpose.

Typical examples:

* OTP state
* temporary uploads
* temporary previews

---

## Delete

The record is permanently removed according to lifecycle rules.

Deletion should be intentional.

---

## Anonymize

Personally identifying ownership is removed while useful aggregate information remains.

Typical examples include some analytics scenarios where product insight remains valuable without identifying an Account.

---

## Regenerate

Some records should not be retained because they can be recreated.

Examples include:

* Render Objects
* derived appearance output
* calculated summaries

---

# 5. Canonical Data Retention

Canonical Product Data should remain available until one of the following occurs:

* user deletes it
* Product Domain removes it
* Account deletion lifecycle applies
* legal or operational policy requires removal

Canonical Product Data should never expire automatically simply because time passed.

---

# 6. System Data Retention

System-owned operational records should remain only while they support the product.

Examples include:

* routing records
* internal identifiers
* lifecycle metadata

Unused operational records should eventually be cleaned according to lifecycle rules.

---

# 7. Temporary Data Retention

Temporary records should have clear expiration behavior.

Examples include:

* OTP verification
* upload processing
* temporary AI context
* temporary preview state

Temporary records must not become permanent because cleanup was forgotten.

---

# 8. AI Suggestion Retention

AI Suggestions should remain only while they provide user value.

Possible outcomes include:

```text
Generated
↓

Accepted

Rejected

Expired
```

Accepted suggestions become Product Domain data.

Rejected suggestions should eventually disappear.

Expired suggestions should not accumulate indefinitely.

Raw AI context should not be retained longer than necessary.

---

# 9. Analytics Retention

Analytics serves observation.

Analytics is not the source of truth.

Raw Events, Aggregates, and Reports may have different retention strategies.

Long-term product insights may remain.

Unnecessary historical detail should not remain forever without purpose.

Retention strategy should balance:

* product value
* storage cost
* performance
* privacy

---

# 10. Event Retention

Events describe what happened.

Not every Event deserves permanent storage.

Events may:

* expire
* aggregate
* archive
* delete

The chosen strategy depends on product value.

Events should never replace canonical Product Data.

---

# 11. Generated Data Retention

Generated data should be evaluated using one question:

> Is storing this cheaper than generating it again?

If regeneration is simple, regeneration should usually be preferred.

Examples include:

* Render Objects
* generated previews
* generated appearance output

---

# 12. Derived Data Retention

Derived Data should remain only while useful.

Whenever practical:

Canonical records

↓

Events

↓

Derived Data

If Derived Data becomes stale, it should be recalculated instead of becoming another source of truth.

---

# 13. Storage Metadata Retention

Structured asset metadata belongs to the Data Platform Service.

It should remain while the corresponding binary asset is meaningful.

Unused metadata should eventually disappear.

Orphaned metadata should not accumulate.

---

# 14. Binary Asset References

Data owns references.

Storage owns binary assets.

If a binary asset disappears, Data should not continue referencing it indefinitely.

Likewise, Storage should not retain binary assets that no structured record can legitimately reference.

Both Platform Services cooperate to prevent orphaned resources.

---

# 15. Account Deletion

Account deletion initiates the retention policy for account-owned information.

Possible outcomes include:

* delete
* archive
* anonymize
* retain only when operationally required

Product Domains define details.

The overall retention policy remains consistent across Minime.

---

# 16. Orphaned Data

Orphaned records should not become normal system state.

Examples include:

* asset metadata without asset
* asset without metadata
* block without owner
* social account without Account
* appearance without Account

Orphaned records should eventually be repaired or removed.

---

# 17. AI Context Retention

AI context should be minimized.

Only information necessary to complete an approved task should remain available during processing.

Long-term storage of prompts, intermediate reasoning, or temporary context should be avoided unless explicitly required.

---

# 18. External Reference Retention

External references should remain only while they are meaningful to the user.

Removing an external reference from Minime does not affect the external platform.

External references are never the external platform itself.

---

# 19. Public Output Retention

Public output is derived from canonical Product Data.

Examples include:

* rendered profile output
* public profile pages
* generated public representations

If canonical data changes, outdated public output should eventually be refreshed or regenerated.

Public output is disposable.

Canonical Product Data is not.

---

# 20. Cleanup Principles

Cleanup should be:

* predictable
* repeatable
* safe
* idempotent

Running cleanup multiple times should never corrupt valid canonical data.

Cleanup should prefer removing unused information instead of risking removal of valid user-owned data.

---

# 21. Cleanup Responsibility

Retention policies belong to the Data Platform Service.

Cleanup execution belongs to implementation.

This document defines **what** should eventually happen.

Implementation decides **how** and **when** it happens.

---

# 22. Cost Awareness

Retention directly affects operating cost.

Every retained record consumes:

* storage
* processing
* backup space
* maintenance effort

Retention decisions should always consider long-term operational cost.

Keeping unnecessary information forever is considered poor architecture.

---

# 23. Privacy Awareness

Retention should respect user privacy.

Information that no longer serves a legitimate purpose should not remain indefinitely.

Where possible, anonymization is preferable to retaining personally identifiable information solely for historical reporting.

---

# 24. Future Policy Changes

Retention periods may change over time.

Changing retention duration should not require redesigning Product Domains.

Retention policy evolves.

Architecture remains stable.

---

# 25. V1 Non-Goals

This policy does not define:

* exact retention durations
* legal compliance requirements
* backup strategy
* disaster recovery
* database cleanup jobs
* scheduled tasks
* storage implementation
* cloud lifecycle rules

Those belong to implementation.

This document defines architectural policy only.

---

# 26. Final Canon

Minime retains information intentionally.

Canonical Product Data is preserved because it represents user-approved truth.

Temporary information expires.

Generated information is regenerated whenever practical.

Derived information is disposable.

Unused information should disappear.

Retention exists to keep Minime simple, efficient, affordable, and trustworthy.

No information should remain in the system without a clear purpose.

No information should disappear without a defined policy.

This is the canonical retention policy for Minime V1.
