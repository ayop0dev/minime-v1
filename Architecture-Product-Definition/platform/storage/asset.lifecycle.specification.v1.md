# Minime Asset Lifecycle Specification V1

**Status:** Canonical
**Version:** V1
**Platform Service:** Storage
**Parent:** `platform/storage/storage.architecture.specification.v1.md`
**Repository State:** Canonical Repository State V1.0

---

# 1. Purpose

This document defines the lifecycle of binary assets managed by the Storage Platform Service.

It explains how binary assets move through Minime from creation to permanent removal.

The goal is to ensure that every binary asset has:

* one identity
* one lifecycle
* one current state
* one clear relationship with structured data

Storage manages binary assets.

Data manages structured references.

Neither responsibility should overlap.

---

# 2. Core Principle

Binary assets are temporary representations of user-approved product data.

The binary asset itself is never the source of truth.

The structured data that references the asset remains the canonical source.

If a binary asset disappears unexpectedly, the product has a problem.

If structured data disappears unexpectedly, the product loses truth.

Storage protects assets.

Data protects meaning.

---

# 3. What Is an Asset?

An Asset is any binary object managed by the Storage Platform Service.

Examples include:

* avatar image
* profile background
* uploaded image
* generated QR image

Assets are binary.

They are not structured records.

---

# 4. Asset Identity

Every asset must have its own identity.

Examples:

```text
asset_id
```

Asset identity must remain stable for the life of the asset.

The asset identity is independent from:

* Account
* Block
* Appearance
* QR Code

Those records reference the asset.

They do not become the asset.

---

# 5. Asset Ownership

Assets inherit ownership from the Product Domain record that references them.

Examples:

```text
Avatar Asset
↓

Profile Content

↓

Account
```

```text
Background Asset
↓

Appearance Configuration

↓

Account
```

Storage does not decide ownership.

Storage respects ownership defined by the Data Platform Service.

---

# 6. Canonical Asset Lifecycle

The default asset lifecycle is:

```text
Requested

↓

Created

↓

Validated

↓

Processed

↓

Stored

↓

Referenced

↓

Served

↓

Replaced (optional)

↓

Unused

↓

Deleted
```

Not every asset requires every state.

The lifecycle provides a common model shared by all asset types.

---

# 7. Requested

An asset begins when a Product Domain requests one.

Examples include:

* user uploads an avatar
* user uploads a background
* system requests a QR image

At this stage, no binary asset is yet considered usable.

---

# 8. Created

The binary asset now exists.

Creation may happen through:

* upload
* generation
* conversion

Creation alone does not make an asset valid.

---

# 9. Validated

The asset should pass basic validation before becoming usable.

Validation may include:

* readable format
* supported type
* acceptable dimensions
* non-corrupted binary content

Implementation defines validation rules.

Storage Architecture defines that validation must exist.

---

# 10. Processed

Storage may prepare the asset for long-term use.

Typical processing may include:

* resizing
* optimization
* format conversion
* metadata extraction
* thumbnail generation (future)

Processing improves storage efficiency and delivery.

Processing does not change product meaning.

---

# 11. Stored

The processed asset becomes part of persistent Storage.

Storage is now responsible for:

* durability
* retrieval
* replacement
* cleanup

Stored does not necessarily mean public.

---

# 12. Referenced

A stored asset becomes useful only after structured data references it.

Example:

```text
Appearance Configuration

↓

background_asset_id

↓

Storage Asset
```

Without a reference, the asset is not part of Product Domain behavior.

---

# 13. Served

The asset is now available for Rendering and Public Profile delivery.

Storage provides binary content.

Rendering decides presentation.

Public Profile delivers it.

Serving does not change the asset.

---

# 14. Replacement

Assets are replaced rather than modified.

Example:

```text
Avatar v1

↓

Avatar v2
```

Replacement is a Data Platform decision. When the Data Platform commits a new canonical asset reference, the previous asset immediately becomes orphaned. Data owns the business reason for replacement. Storage does not independently decide that an asset has been superseded.

Storage detects the orphaned asset and executes its physical deletion. Replaced assets do not remain in storage. There is no archive of replaced asset versions. Minime V1 does not retain replaced assets for history, rollback, or media-library access.

---

# 15. Unused

An asset becomes unused when no active canonical record in the Data Platform references it.

The Data Platform determines whether an asset is referenced. Storage does not independently assess asset relevance to decide it is obsolete.

Examples:

* avatar replaced
* background removed
* QR regenerated

Unused assets should not remain forever.

They become candidates for cleanup.

---

# 16. Deleted

Deletion permanently removes the binary asset.

Deletion should occur only after confirming that:

* no active references remain
* lifecycle rules permit removal

Storage must never remove assets still required by canonical Product Domain data.

---

# 17. Asset References

Assets are never consumed directly by Product Domains.

Product Domains reference structured asset metadata.

Storage resolves those references into binary content.

This separation keeps Product Domains independent from Storage implementation.

---

# 18. Shared Assets

Minime V1 assumes assets are normally owned by one Account.

Sharing binary assets between unrelated Accounts is outside V1 scope.

Each Account should manage its own assets independently.

---

# 19. Generated Assets

Generated assets follow the same lifecycle.

Examples include:

* QR images
* future generated media

Generation replaces upload.

Everything else remains the same.

Generated assets are not special ownership objects.

---

# 20. Temporary Assets

Temporary assets exist only during processing.

Examples include:

* upload buffers
* intermediate conversions
* temporary previews

Temporary assets should never remain after processing completes.

---

# 21. Orphaned Assets

An orphaned asset has no valid canonical record in the Data Platform referencing it.

Orphan status is determined by the Data Platform — when no canonical data record references an asset, that asset is considered orphaned. Storage does not independently assess business lifecycle to determine obsolescence.

Examples include:

* replaced avatar
* deleted background
* failed upload left behind

Orphaned assets should eventually be cleaned safely.

Storage should never accumulate orphaned assets indefinitely.

---

# 22. Failed Assets

Not every asset successfully reaches Stored state.

Failures may occur during:

* upload
* validation
* processing

Failed assets should not become referenced.

Failed temporary artifacts should eventually disappear.

---

# 23. Asset Consistency

Storage should guarantee:

* one identity
* one current lifecycle state
* valid references
* predictable replacement
* safe cleanup

An asset should never simultaneously be:

```text
Referenced

and

Deleted
```

Lifecycle consistency is mandatory.

---

# 24. Asset State Transitions

State transitions should always move forward.

Typical examples:

```text
Requested
↓

Created
↓

Validated
↓

Processed
↓

Stored
```

or

```text
Stored

↓

Referenced

↓

Served
```

or

```text
Referenced

↓

Unused

↓

Deleted
```

Storage should avoid unnecessary backward transitions.

---

# 25. Lifecycle Events

Meaningful lifecycle transitions may emit Events.

Examples include:

```text
Asset Uploaded

Asset Validated

Asset Stored

Asset Replaced

Asset Deleted
```

Events describe lifecycle transitions.

They do not perform lifecycle transitions.

---

# 26. Lifecycle Guarantees

Every asset should guarantee:

* stable identity
* explicit ownership through Data
* predictable lifecycle
* safe replacement
* safe deletion

Every asset should eventually reach a final lifecycle outcome.

No asset should remain permanently in intermediate states.

---

# 27. V1 Non-Goals

This specification does not define:

* image optimization algorithms
* upload APIs
* CDN behavior
* storage providers
* filesystem layouts
* object storage services
* media transcoding pipelines
* version history

Those belong to implementation.

---

# 28. Final Canon

Binary assets exist to support user-approved Product Domain data.

Storage owns the lifecycle of those assets.

Data owns the structured references.

Product Domains own why the asset exists.

Rendering consumes prepared assets.

Public Profile delivers them.

Every asset should have one identity, one lifecycle, one current state, and one clear relationship with structured data.

No binary asset should remain in Minime without a purpose.

No binary asset should disappear while still required by canonical data.

This is the canonical asset lifecycle for Minime V1.
