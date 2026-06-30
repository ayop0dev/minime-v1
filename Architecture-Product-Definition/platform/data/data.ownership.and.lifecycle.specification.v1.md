# Minime Data Ownership and Lifecycle Specification V1

**Status:** Canonical
**Version:** V1
**Platform Service:** Data
**Parent:** `platform/data/data.architecture.specification.v1.md`
**Repository State:** Canonical Repository State V1.0

---

# 1. Purpose

This document defines the ownership and lifecycle rules for canonical data in Minime V1.

It explains:

* who owns data
* how ownership is determined
* how ownership changes
* how records move through their lifecycle
* how Product Domains should apply these rules consistently

This document defines shared rules.

Individual Product Domains may specialize them, but they must never contradict them.

---

# 2. Core Principle

Every durable record in Minime must answer two questions.

```text
Who owns me?

How do I live?
```

If either answer is unclear, the record is not architecturally complete.

---

# 3. Ownership Principles

Ownership is one of the foundations of Minime.

Ownership is never guessed.

Ownership is never inferred.

Ownership is always explicit.

Every durable record must have exactly one ownership model.

---

## One Owner

A record may have:

* one owner
* or no owner if it is explicitly system-owned.

A record must never have multiple owners.

If multiple Accounts need access to the same information, ownership still belongs to one Account.

Sharing is an access concern.

Ownership is a data concern.

---

## Ownership Is Stable

Ownership should remain stable throughout the life of a record.

Changing ownership should be extremely rare.

Most ownership changes indicate that a new record should be created instead.

---

## Ownership Is Independent From Identity

Ownership identifies who owns the record.

Identity identifies the record itself.

Example:

```text
block_id

≠

account_id
```

These concepts must never be merged.

---

# 4. Ownership Models

Minime V1 recognizes four ownership models.

---

## Account-Owned

Most Product Domain entities belong here.

Examples:

* Profile Content
* Blocks
* Appearance
* Social Accounts
* Out Links
* QR Codes

Ownership reference:

```text
account_id
```

---

## System-Owned

Some entities belong to Minime itself.

Examples:

* Social Platform Definitions
* System Theme Definitions
* Reserved Usernames

These entities are maintained by the system.

They are never owned by individual Accounts.

---

## Generated

Some records are generated from other records.

Examples:

* Render Objects
* Generated QR images
* Derived analytics

Generated records inherit context.

They do not become ownership roots.

---

## Temporary

Temporary records exist only while completing a process.

Examples:

* Username reservation state
* Upload processing
* Temporary AI context

Temporary ownership exists only for the duration of the process.

Temporary records must expire.

---

# 5. Lifecycle Principles

Every meaningful record follows a lifecycle.

A lifecycle defines:

* how a record is created
* when it becomes active
* how it changes
* how it ends

No canonical record should exist without lifecycle rules.

---

## Canonical Lifecycle

The default lifecycle is:

```text
Created

↓

Active

↓

Updated

↓

Disabled (optional)

↓

Archived (optional)

↓

Deleted
```

Not every entity needs every state.

Product Domains may simplify this lifecycle.

They may not invent contradictory behavior.

---

# 6. Record Creation

A record may be created only by:

* user action
* approved Product Domain workflow
* required system operation
* approved import
* approved generation process

AI alone may not create canonical user-owned records.

External platforms may not create canonical user-owned records.

Analytics may not create canonical user-owned records.

---

# 7. Record Activation

A record becomes active only after meeting Product Domain requirements.

Examples include:

* validation completed
* user approval received
* creation process finished

Activation should be explicit.

Not assumed.

---

# 8. Record Updates

Updates preserve identity.

Updates preserve ownership.

Updates change state.

They do not replace the record unless the Product Domain explicitly defines replacement behavior.

Updates must remain traceable to:

* user action
* approved workflow
* required system operation

AI may recommend updates.

Only approved workflows may apply them.

---

# 9. Record Replacement

Sometimes replacing a record is preferable to mutating it.

Examples include:

* replacing an uploaded image
* replacing a generated QR asset
* replacing temporary processing metadata

Replacement should preserve ownership.

Old records should follow lifecycle rules instead of remaining orphaned.

---

# 10. Record Archival

Archival keeps a record without treating it as active.

Archived records:

* are no longer operational
* remain historically meaningful where appropriate
* should not appear in active product behavior

Not every Product Domain needs archival.

---

# 11. Record Deletion

Deletion ends the lifecycle of a record.

Deletion behavior belongs to Product Domains.

The Data Platform defines only the shared principles.

Deletion must always be intentional.

Deletion must never leave undefined ownership.

Deletion must never silently break relationships.

## Hard Delete vs. Soft Delete

Account-owned product data uses Hard Delete by default. Hard Delete permanently removes the record and cannot be reversed.

Soft Delete — retaining a record with a deleted status — is used only when a domain specification explicitly defines an archival window before permanent removal. Out Links use Soft Delete during the 90-day archival window before purge. No other V1 entity uses Soft Delete unless its domain specification explicitly defines it.

When a domain specification does not define the deletion type, Hard Delete applies.

---

# 12. Cascade Rules

Some records own other records indirectly.

Example:

```text
Account

↓

Blocks

↓

Block Style Overrides
```

Deleting the parent requires explicit rules for dependents.

Every Product Domain must define one of the following behaviors:

* delete
* archive
* anonymize
* detach
* preserve

Undefined cascading is not allowed.

---

# 13. Promotion

Some records move between trust levels.

Examples:

```text
AI Suggestion

↓

User Approval

↓

Canonical Product Data
```

```text
Temporary Upload

↓

Stored Asset Metadata
```

```text
Reserved Username

↓

Active Username
```

Promotion must always be:

* explicit
* traceable
* approved by Product Domain rules

No automatic promotion into User-Owned Data is allowed.

---

# 14. Demotion

Records may also move to lower operational states.

Examples:

```text
Active

↓

Disabled
```

```text
Active

↓

Archived
```

```text
Published

↓

Unpublished
```

Demotion changes availability.

It does not change ownership.

---

# 15. Ownership Transfer

Ownership transfer is intentionally restricted.

Most records should never change owner.

If ownership must change, Product Domains should prefer:

```text
Create New Record

↓

Transfer Meaning

↓

Retire Old Record
```

instead of mutating ownership.

---

# 16. Account Deletion

Account deletion is the highest ownership event.

It ends the lifecycle of the ownership root.

Account-owned records should follow explicit cascade rules.

Examples include:

* Blocks
* Profile Content
* Appearance
* Social Accounts
* Out Links
* Stored Asset Metadata

Each Product Domain may define details.

None may ignore Account deletion.

---

# 17. System-Owned Records

System-owned records do not disappear when an Account is deleted.

Examples include:

* supported social platforms
* available themes
* reserved system usernames

These belong to Minime.

Not to users.

---

# 18. Generated Records

Generated records depend on canonical records.

Examples include:

* Render Objects
* Generated QR assets
* Analytics aggregates

If canonical source data changes, generated records should be refreshed or regenerated.

Generated records must never become the source of truth.

---

# 19. Temporary Records

Temporary records should be short-lived.

They should:

* expire
* complete
* promote
* or disappear

Temporary records must not silently become permanent.

---

# 20. Relationship Ownership

Relationships are owned by the record that declares them.

A relationship must remain valid while both records exist.

Broken relationships should never become normal system state.

If a referenced record disappears, the Product Domain must define the expected behavior.

---

# 21. Lifecycle Consistency

All Product Domains should describe lifecycle using the same language whenever possible.

Preferred lifecycle vocabulary:

* Created
* Active
* Updated
* Disabled
* Archived
* Deleted

Avoid inventing different words for the same concept.

Consistency improves maintainability.

---

# 22. Lifecycle Events

Meaningful lifecycle transitions may emit Events.

Examples include:

```text
Block Created

Profile Published

QR Generated

Username Changed
```

Events describe transitions.

They do not perform transitions.

---

# 23. Lifecycle and Storage

Deleting a data record does not automatically define what happens to binary assets.

Storage owns binary lifecycle.

Data owns structured lifecycle.

Both Platform Services must cooperate.

Neither replaces the other.

---

# 24. Lifecycle and Rendering

Rendering consumes active canonical records.

Archived, deleted, rejected, expired, or disabled records must not appear unless a Product Domain explicitly allows them.

Rendering never revives inactive records.

---

# 25. Lifecycle and AI

AI Suggestions have their own lifecycle.

Typical flow:

```text
Generated

↓

Presented

↓

Accepted
Edited
Rejected
Expired
```

Acceptance promotes user-approved information into Product Domain data.

Rejection ends the suggestion lifecycle.

---

# 26. Lifecycle and Analytics

Analytics observes lifecycle transitions.

Analytics does not define them.

Analytics may report:

* created records
* updated records
* deleted records

It must never control lifecycle decisions.

---

# 27. Lifecycle and Public Profile

Public Profile serves only active, approved, renderable data.

Inactive records remain part of system history where appropriate.

They are not part of the visitor experience.

---

# 28. Lifecycle Guarantees

Every canonical entity should guarantee:

* one clear owner
* one clear identity
* one clear lifecycle
* one current state

A record should never exist in contradictory states.

Example:

```text
Archived

and

Active
```

at the same time.

---

# 29. V1 Non-Goals

This specification does not define:

* workflow engines
* approval engines
* version control systems
* audit systems
* event sourcing
* multi-owner collaboration
* organization ownership
* workspace ownership

Those are outside V1.

---

# 30. Final Canon

Ownership answers:

> Who is responsible for this record?

Lifecycle answers:

> What happens to this record over time?

Every canonical record in Minime V1 must have:

* one identity
* one ownership model
* one lifecycle
* one current state

Ownership must always be explicit.

Lifecycle must always be defined.

Neither may depend on assumptions.

This guarantees that every Product Domain behaves consistently while remaining free to define its own business rules within the shared data architecture.
