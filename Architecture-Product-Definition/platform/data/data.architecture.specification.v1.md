# Minime Data Architecture Specification V1

**Status:** Canonical
**Version:** V1
**Platform Service:** Data
**Parent:** `platform/platform.architecture.specification.v1.md`
**Repository State:** Canonical Repository State V1.0

---

# 1. Purpose

This document defines the Data Architecture of Minime V1.

The Data Platform Service exists to define how structured information is owned, classified, persisted, related, retained, and deleted across Minime.

It does not choose a database technology.

It does not define implementation schemas.

It does not replace Product Domain specifications.

Its job is to provide one shared data model philosophy for all Product Domains.

---

# 2. Core Principle

Minime data architecture is built around one rule:

> The user is the source of truth.

Any data that becomes part of a public Minime profile must come from the user, be confirmed by the user, or be explicitly approved by the user.

External platforms, AI, analytics, system-generated suggestions, imported values, and inferred information may support the user.

They never replace the user.

---

# 3. What the Data Platform Service Owns

The Data Platform Service owns the shared rules for structured data.

It owns:

* data classification
* canonical entity rules
* ownership rules
* relationship rules
* lifecycle rules
* retention rules
* deletion rules
* structured metadata rules
* system-generated record rules

The Data Platform Service defines how data should be treated.

It does not define what the product means.

That remains the responsibility of Product Domains.

---

# 4. What the Data Platform Service Does Not Own

The Data Platform Service does not own:

* Product Domain behavior
* business validation
* profile layout
* block meaning
* appearance rules
* rendering logic
* public profile serving
* analytics interpretation
* AI decisions
* binary file storage
* storage optimization
* event transport
* infrastructure choices

The Data Platform Service supports these areas by keeping structured data consistent.

It does not control them.

---

# 5. Data Is Not the Product

Data is the durable representation of product decisions.

The Product Domain decides what something means.

The Data Platform Service decides how the approved structured record is maintained.

For example:

* The Profile domain defines what a Button Block means.
* The Data Platform Service defines how the Button Block record follows ownership, identity, lifecycle, and deletion rules.

The Data Platform Service must never absorb Product Domain meaning.

---

# 6. Data Categories

Minime V1 recognizes the following data categories.

These categories define how data should be handled across the system.

---

## 6.1 User-Owned Data

User-Owned Data is information created, entered, arranged, approved, or explicitly confirmed by the user.

Examples include:

* account identity fields
* username
* display name
* bio
* profile content
* blocks
* button labels
* button URLs
* social account entries
* appearance choices
* published profile structure

User-Owned Data is the highest-trust data category in Minime.

It may appear publicly if Product Domain rules allow it.

It must not be silently changed by AI, Analytics, external platforms, or automated systems.

---

## 6.2 System-Owned Data

System-Owned Data is information created by Minime to operate the product.

Examples include:

* internal record identifiers
* timestamps
* status fields
* ownership references
* routing identifiers
* QR code records
* out-link routing records
* internal lifecycle state

System-Owned Data supports the product.

It does not replace User-Owned Data.

It should be hidden from visitors unless explicitly transformed into a public-safe output by Rendering.

---

## 6.3 Generated Data

Generated Data is created by Minime based on existing user-approved data or system behavior.

Examples include:

* generated QR codes
* generated short out-link references
* generated render objects
* generated slugs or routing values
* generated previews
* AI-generated suggestions before approval

Generated Data may support the user experience.

It is not automatically User-Owned Data.

Generated Data becomes User-Owned Data only when the relevant Product Domain and user approval rules say so.

---

## 6.4 Derived Data

Derived Data is calculated from existing data.

Examples include:

* analytics totals
* view counts
* click counts
* computed display state
* resolved appearance values
* profile completeness indicators
* aggregated reports

Derived Data may be useful.

It is not source data.

If Derived Data is lost, it should usually be possible to recalculate it from canonical records or events, unless a Product Domain explicitly defines otherwise.

---

## 6.5 Temporary Data

Temporary Data exists only to complete a short-lived process.

Examples include:

* username reservation state
* upload processing state
* temporary AI suggestion context
* temporary preview state
* short-lived validation state

Temporary Data must have a clear expiration rule.

Temporary Data should not be retained indefinitely.

Temporary Data must not become canonical unless explicitly promoted through a Product Domain rule.

---

## 6.6 External Reference Data

External Reference Data points to something outside Minime.

Examples include:

* social platform identifiers
* social URLs
* external website URLs
* connected account references
* future OAuth provider references

External Reference Data is not proof of external truth.

For V1, external references are user-confirmed or user-entered.

Minime may store the reference.

Minime must not claim the external account is true, verified, owned, active, or accurate unless the relevant Product Domain explicitly defines a verification process.

---

## 6.7 AI Suggestion Data

AI Suggestion Data is output generated by AI before the user approves it.

Examples include:

* suggested username ideas
* suggested bio improvements
* suggested button titles
* suggested profile improvements
* onboarding recommendations

AI Suggestion Data is never automatically User-Owned Data.

It must remain separate from canonical user data until the user approves, edits, or accepts it.

Rejected AI suggestions should not become part of canonical product records.

---

# 7. Canonical Data

Canonical Data is the official structured data that Minime relies on as the current truth for a given product concept.

A record is canonical only when:

* it belongs to a defined Product Domain
* it has a clear owner
* it follows the domain's validation rules
* it follows Data Platform lifecycle rules
* it is stored as the current accepted version of that concept

Canonical Data should be stable, queryable, and lifecycle-aware.

Not all stored data is canonical.

Temporary data, raw suggestions, logs, cached values, and intermediate processing state are not canonical.

---

# 8. Canonical Entity

A Canonical Entity is a durable structured record representing a core product or system concept.

Examples may include:

* Account
* Connected Account
* Profile Content record
* Block
* Appearance configuration
* Social Account
* Out Link
* Analytics Event
* QR Code
* Stored Asset Metadata
* AI Suggestion record

This document does not finalize the full entity map.

The complete entity list belongs in:

```text
platform/data/canonical.entities.map.v1.md
```

This file defines the rules those entities must follow.

---

# 9. Entity Identity

Every Canonical Entity must have a stable record identity.

A record identity must:

* identify the record itself
* remain stable for the life of the record
* not be confused with ownership
* not be reused for another record

Record identity and ownership are different concepts.

For example:

```text
block_id       = identity of the block record
account_id     = owner of the block record
```

A record must not use `account_id` as its own record identity.

`account_id` identifies ownership.

It does not identify every owned record.

---

# 10. Ownership

Ownership defines which Account owns a record.

In Minime V1, owned records use `account_id` as the ownership reference.

The account is the owner boundary for V1.

Owned records must be traceable back to an Account unless a Product Domain explicitly defines a system-level record that is not account-owned.

Ownership must be clear.

Ownership must not be inferred from URLs, usernames, public handles, or external platform identifiers.

---

# 11. No Persisted Profile Entity

Minime V1 does not treat Profile as a separate persisted owner entity.

Profile is a public expression of Account-owned data.

There is no `profile_id` ownership model in V1.

Owned resources must reference `account_id`, not `profile_id`.

This preserves the existing Identity Canon and avoids splitting ownership across Account and Profile.

---

# 12. Relationships

Data relationships must be explicit.

A relationship should answer:

* which record owns this record?
* which record does this record reference?
* can the referenced record be deleted?
* what happens when the owner is deleted?
* is the relationship required or optional?

Relationships must not depend on hidden assumptions.

Relationships must not be created only through naming conventions.

If a record depends on another record, the dependency must be visible in the relevant Product Domain or canonical entity map.

---

# 13. Data Lifecycle

Every meaningful data record should have a lifecycle.

A lifecycle defines how the record moves from creation to removal.

A simple V1 lifecycle may include:

```text
Created
↓
Active
↓
Updated
↓
Disabled / Archived / Deleted
```

Not every record needs every state.

The Product Domain defines the meaningful states.

The Data Platform Service defines the expectation that records must not live forever without a lifecycle rule.

---

# 14. Creation

Data creation happens when a Product Domain accepts valid input or when the system creates a required supporting record.

User-created data must pass Product Domain validation before becoming canonical.

System-created data must have a clear purpose.

AI-created data must remain a suggestion until user approval.

External reference data must remain a reference unless the Product Domain defines verification.

---

# 15. Updates

Updates must preserve ownership.

Updating a record must not silently move it from one Account to another.

Updates must respect Product Domain rules.

Updates must not allow Platform Services to override business meaning.

AI may propose an update.

Analytics may reveal a reason to update.

Only the user or an approved Product Domain workflow may apply the update.

## Concurrency

Minime V1 resolves concurrent writes to the same record using Last Write Wins.

The last write to complete becomes the current state. The system does not attempt to merge concurrent changes.

This is the canonical concurrency model for the entire platform. Product Domains must follow it. No Product Domain may introduce an alternative concurrency strategy. Any change to this model requires revision of the Platform Architecture itself.

---

# 16. Deletion

Deletion must be intentional and lifecycle-aware.

When data is deleted, the system must know whether the deletion is:

* soft deletion
* hard deletion
* archival
* expiration
* replacement cleanup

Product Domains may define what deletion means for their own records.

The Data Platform Service defines that deletion must not be accidental, hidden, or undefined.

---

# 17. Account Deletion

Account deletion is the highest-level deletion event in V1.

When an Account is permanently removed, all account-owned canonical records should either be:

* deleted
* anonymized
* retained only if legally or operationally required
* retained only as non-user-identifying aggregate data

The specific cascade rules belong in:

```text
platform/data/data.ownership.and.lifecycle.specification.v1.md
```

No Product Domain should invent its own account deletion cascade independently.

---

# 18. Retention

Retention defines how long data remains stored.

Minime V1 must avoid indefinite retention of data without a reason.

Retention rules are especially important for:

* analytics events
* temporary verification state
* AI suggestion history
* deleted records
* upload processing metadata
* logs or audit-like records

The specific retention policy belongs in:

```text
platform/data/data.retention.policy.v1.md
```

This document defines the principle.

Retention must be intentional.

---

# 19. Data and Storage Boundary

The Data Platform Service stores structured data and metadata.

The Storage Platform Service stores binary assets.

Data may reference Storage.

Storage may provide asset references.

But binary files must not be stored as structured Data records.

For example:

```text
image_id
avatar_asset_id
background_asset_id
qr_asset_id
```

These are Data references.

The actual binary file belongs to Storage.

When the Data Platform updates a canonical asset reference, the previous asset becomes orphaned. This is a Data Platform business decision. Storage is responsible for detecting orphaned assets and executing their physical deletion. Data owns the decision; Storage performs the deletion.

---

# 20. Data and Events Boundary

Data and Events are related but different.

Data represents current or retained structured state.

Events describe something that happened.

For example:

```text
Button Block record = Data
Button Clicked = Event
```

Events may create analytics data.

Events may be retained for reporting.

But Events should not replace canonical product records.

A product record should not be reconstructed from events unless a future architecture explicitly allows event sourcing.

Minime V1 is not event-sourced.

---

# 21. Data and AI Boundary

AI may read allowed context.

AI may generate suggestions.

AI may explain improvements.

AI must not directly mutate canonical data.

AI output must remain separate from canonical data until the user accepts, edits, or approves it.

The AI Platform Service may store suggestion records when needed.

Those records must be classified as AI Suggestion Data, not User-Owned Data.

---

# 22. Data and Rendering Boundary

Rendering consumes prepared data.

Rendering does not own data.

Rendering does not create source truth.

Rendering does not persist Product Domain changes.

Rendering may produce Render Objects or read-safe output.

Those outputs are prepared representations.

They are not the canonical source records.

If rendered output becomes stale, it should be regenerated from canonical data.

---

# 23. Data and Analytics Boundary

Analytics consumes Events and may produce Derived Data.

Analytics does not own Product Domain behavior.

Analytics does not change canonical profile data.

Analytics does not decide what users should publish.

Analytics may report:

* views
* clicks
* trends
* summaries
* aggregated behavior

Analytics may inform the user.

It must not control the product.

---

# 24. Data Trust Levels

Not all data has the same trust level.

Minime V1 recognizes these trust levels:

```text
User-approved data
System-owned operational data
Generated data
Derived data
Temporary data
External reference data
AI suggestion data
```

When in doubt, lower-trust data must not overwrite higher-trust data.

AI suggestion data must not overwrite User-Owned Data.

External reference data must not overwrite User-Owned Data.

Derived analytics must not overwrite Product Domain decisions.

---

# 25. Data Promotion

Data promotion means moving information from a lower-trust category into a higher-trust category.

Examples:

```text
AI suggestion
↓ user approves
User-Owned Data
```

```text
Temporary upload state
↓ upload completes
Stored Asset Metadata
```

```text
External social URL
↓ user confirms
User-Owned Social Account Reference
```

Promotion must be explicit.

Promotion must be traceable.

Promotion must respect Product Domain rules.

No background process may silently promote data into User-Owned Data.

---

# 26. Data Minimization

Minime should store only what it needs.

The system should avoid storing:

* unnecessary external platform data
* raw AI context longer than needed
* duplicated binary files as structured data
* visitor-identifying information unless required
* temporary data without expiration
* unapproved suggestions indefinitely

Data minimization keeps the system simpler, cheaper, safer, and easier to maintain.

---

# 27. Visitor Data

Visitors are not Account owners.

Visitor interactions may create Events.

Visitor interactions may contribute to Analytics.

Visitor data should be minimal.

In V1, Analytics is focused on views and clicks, not deep visitor identity.

Visitor data must not become Profile Content.

Visitor data must not become User-Owned Data.

---

# 28. External Platform Data

External platforms may provide references or context.

But Minime V1 must not treat external platform data as truth.

For Social Accounts:

* the user may enter or approve a social account reference
* Minime may store that reference
* Minime must not claim external ownership or verification unless explicitly defined

The retired Discovery Engine must not return as a data source.

Social account data in V1 is user-confirmed reference data.

---

# 29. AI Data

AI data must be handled carefully.

AI may produce useful output.

But AI output is not truth.

AI output must not become canonical unless the user accepts it.

AI context should be limited to the minimum information needed for the task.

AI suggestions should be stored only when needed for user experience, approval flow, or future review.

Raw prompts, unnecessary context, and rejected suggestions should not be retained without a defined reason.

---

# 30. Metadata

Metadata is structured information about records.

Examples include:

* created_at
* updated_at
* deleted_at
* status
* sort_order
* owner reference
* source type
* lifecycle state

Metadata helps the system operate consistently.

Metadata must not be confused with product content.

For example:

```text
button title = product content
button sort_order = metadata
```

Both may be stored.

But they mean different things.

## Canonical Time

All timestamps in Minime V1 are stored and compared in Coordinated Universal Time (UTC).

Expiration calculations, retention decisions, and lifecycle scheduling use UTC.

Display formatting is a presentation concern and does not affect stored timestamp values.

---

# 31. Status Fields

Status fields should be clear and domain-appropriate.

Examples may include:

* active
* disabled
* deleted
* archived
* expired
* pending
* rejected

A status should not be vague.

A status should not mean different things in different contexts unless clearly scoped.

Product Domains define their statuses.

The Data Platform Service requires that statuses be understandable and lifecycle-aware.

---

# 32. Sorting and Ordering

Some records require ordering.

Examples include:

* blocks
* social icons
* profile sections
* buttons

Ordering data should be explicit.

Ordering must not rely on creation time unless the Product Domain defines that rule.

Common ordering metadata may include:

```text
sort_order
position
display_order
```

The exact field name should be consistent inside each Product Domain.

---

# 33. Read Models

A Read Model is a prepared shape of data optimized for reading or presentation.

Render Objects are a form of read-safe output.

Read Models may be derived from canonical data.

They are not the original source of truth.

If a Read Model becomes stale, it should be refreshed from canonical data.

Product Domains and Rendering may consume Read Models.

But canonical ownership remains with the original data records.

## Generated Artifacts

The following are generated or derived artifacts and must not be treated as the Source of Truth:

* rendered HTML and rendered public page output
* generated metadata, JSON-LD, sitemap fragments, and robots output
* cache entries
* generated QR variants (such as PNG representations)
* AI suggestions and AI analysis results
* analytics aggregations and derived reporting views

Generated artifacts are owned by the system layer that produces them. A Product Domain consuming a generated artifact does not become its owner. Generated artifacts never become canonical business data.

Generated artifacts may always be deleted and regenerated. The Source of Truth remains: user-entered data, active uploaded assets, and canonical platform data records.

---

# 34. Write Rules

Only approved workflows may write canonical data.

A write must be traceable to one of the following:

* user action
* Product Domain workflow
* system lifecycle operation
* approved import or confirmation flow
* explicit user approval of a suggestion

A write must not be traceable only to:

* AI inference
* analytics observation
* external platform assumption
* visitor behavior
* rendering output

---

# 35. Read Rules

Different parts of Minime may read data for different reasons.

Product Domains may read data to enforce rules.

Rendering may read prepared data to create visitor-safe output.

Analytics may read Events and Derived Data.

AI may read allowed context to generate suggestions.

Public Profile may read rendered output to serve visitors.

Every reader must respect the boundary of the data it reads.

Reading data does not grant ownership over it.

---

# 36. Data Consistency

Data should remain internally consistent.

Consistency means:

* ownership references are valid
* required relationships are present
* deleted records are not treated as active
* stale derived outputs can be refreshed
* public output reflects approved data
* temporary data does not become permanent accidentally

The Data Platform Service supports consistency.

Product Domains define the meaning of consistency for their own records.

---

# 37. Data Errors

Data errors should be handled safely.

Examples include:

* missing record
* deleted owner
* invalid relationship
* stale reference
* expired temporary data
* missing asset metadata

Internal data errors must not expose private implementation details to visitors.

Public-facing behavior should remain safe, generic, and user-friendly.

---

# 38. V1 Non-Goals

The Data Platform Service does not introduce:

* a chosen database engine
* SQL schema design
* ORM models
* migrations
* event sourcing
* data warehouse architecture
* advanced permissions
* multi-account collaboration
* complex audit history
* search indexing
* caching architecture
* BI pipelines
* external data enrichment
* autonomous AI data mutation

These may be considered later only if a real product need appears.

They are not part of V1 Data Architecture.

---

# 39. Required Follow-Up Files

This document is the parent Data Architecture specification.

The following Data Platform files should define narrower details:

```text
platform/data/canonical.entities.map.v1.md
platform/data/data.ownership.and.lifecycle.specification.v1.md
platform/data/data.retention.policy.v1.md
```

These files must follow the principles defined here.

They must not contradict:

* user as source of truth
* `account_id` ownership
* no persisted `profile_id`
* AI suggestion-only model
* Storage owns binary assets
* Events describe what happened
* Rendering owns presentation preparation only

---

# 40. Final Canon

Minime V1 data architecture is simple:

The user provides or approves truth.

Product Domains define what that truth means.

The Data Platform Service maintains structured records and metadata.

Storage owns binary assets.

Events describe what happened.

Analytics observes events.

AI suggests improvements.

Rendering prepares visitor-safe output.

No system component may silently replace the user's decisions.

No data responsibility should exist without a clear owner.

No lower-trust data may overwrite higher-trust data without explicit user-approved promotion.

This is the canonical data foundation for Minime V1.
