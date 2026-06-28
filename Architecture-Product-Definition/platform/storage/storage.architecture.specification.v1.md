# Minime Storage Architecture Specification V1

**Status:** Canonical
**Version:** V1
**Platform Service:** Storage
**Parent:** `platform/platform.architecture.specification.v1.md`
**Repository State:** Canonical Repository State V1.0

---

# 1. Purpose

This document defines the Storage Platform Service for Minime V1.

The Storage Platform Service is responsible for managing the complete lifecycle of binary assets used throughout Minime.

Storage exists to make binary assets:

* safe
* efficient
* reusable
* inexpensive
* easy to serve
* easy to replace
* easy to remove

Storage never owns product meaning.

Storage owns binary assets only.

---

# 2. Core Principle

Storage owns files.

Data owns structured information.

These responsibilities must never be mixed.

For example:

```text
Avatar Image
```

The binary image belongs to Storage.

The structured reference to that image belongs to Data.

Storage and Data cooperate.

Neither replaces the other.

---

# 3. What Storage Means

Storage is not a folder.

Storage is not a cloud provider.

Storage is not an upload endpoint.

Storage is the Platform Service responsible for the entire lifecycle of binary assets.

This includes:

* receiving
* validating
* processing
* storing
* replacing
* serving
* cleaning
* deleting

Storage defines *what happens to files*.

Implementation decides *where files live*.

---

# 4. What Storage Owns

The Storage Platform Service owns binary assets.

Examples include:

* user-uploaded images
* profile background images
* generated QR assets

Storage also owns:

* asset lifecycle
* asset processing
* asset optimization
* asset replacement
* asset cleanup
* asset delivery preparation

Storage does not own the structured metadata describing those assets.

That belongs to the Data Platform Service.

Storage does not classify binary objects by their product purpose. Names such as "avatar" or "background" belong to Product Domains and the Data Platform. Storage manages the binary object only.

---

# 5. What Storage Does Not Own

Storage intentionally does **not** own:

* Product Domain behavior
* structured data
* canonical entities
* ownership rules
* rendering logic
* analytics
* AI suggestions
* routing
* profile content
* appearance meaning

Storage is concerned only with binary assets.

---

# 6. Binary Assets

A binary asset is any file whose value is its binary content rather than structured fields.

Examples include:

* JPG
* PNG
* WebP
* SVG
* generated QR images

Binary assets are never canonical Product Domain data.

They support Product Domain data.

---

# 7. Asset Lifecycle

Every binary asset follows a lifecycle.

Typical lifecycle:

```text
Requested
↓

Uploaded / Generated
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

Not every asset requires every stage.

The Storage Platform Service defines the lifecycle.

Implementation defines how each stage executes.

---

# 8. Storage Principles

Storage follows several simple principles.

---

## Storage Is Independent

Storage should not know why a file exists.

It only knows how to manage it safely.

---

## Storage Is Reusable

The same Storage Platform Service should support multiple Product Domains.

Storage must never become specific to:

* Blocks
* Appearance
* QR
* Social Accounts

Those domains simply consume Storage.

---

## Storage Minimizes Waste

Storage should avoid:

* duplicate files
* orphaned files
* unnecessary copies
* oversized assets
* forgotten temporary uploads

Unused binary assets should eventually disappear.

---

## Storage Is Replaceable

Changing storage technology must not require Product Domain redesign.

Product Domains reference assets.

Storage decides how those assets are stored.

---

## Storage Object Identity

Storage object identifiers — including paths, filenames, keys, and object names — carry no business meaning.

Business-meaningful names such as avatar, background, block-image, qr, logo, or profile-media must never appear inside Storage object identifiers to signal purpose. Storage never interprets object identifiers to infer product meaning.

The Data Platform owns what an object represents, where it is used, its current asset relationship, and its lifecycle meaning.

Storage owns the technical identifier used to locate the binary object, object persistence, object retrieval, and object deletion.

No implementation should require reading business meaning from a Storage object identifier.

---

# 9. Storage and Data

Storage and Data work together.

```text
Data

↓

Asset Reference

↓

Storage

↓

Binary Asset
```

The Data Platform Service owns:

* asset identity
* ownership reference
* metadata
* lifecycle reference

The Storage Platform Service owns:

* binary content
* processing
* optimization
* physical storage
* cleanup

Neither service should duplicate the other's responsibility.

---

# 10. Storage and Product Domains

Product Domains request Storage when binary assets are required.

Examples include:

* Profile Content requesting an avatar
* Appearance requesting a background image
* QR Code requesting a generated QR asset

Storage never decides whether the Product Domain should use an asset.

It only manages the asset itself.

---

# 11. Storage and Rendering

Rendering consumes binary assets prepared by Storage.

Rendering never stores assets.

Rendering never processes uploads.

Rendering receives assets that are already ready for visitor consumption.

If an asset changes, Rendering should simply consume the new asset reference.

---

# 12. Storage and Public Profile

Public Profile serves assets managed by Storage.

Public Profile should never become responsible for:

* asset optimization
* asset processing
* asset cleanup

Its responsibility is delivery.

Storage prepares.

Public Profile serves.

---

# 13. Storage and AI

AI may request binary assets as input where Product Domain rules allow.

Examples:

* profile image suggestions
* image-aware recommendations (future)

AI never owns stored assets.

AI must never silently replace user assets.

AI output becomes a stored asset only through approved Product Domain workflows.

---

# 14. Storage and Events

Storage operations may emit Events.

Examples include:

```text
Asset Uploaded

Asset Replaced

Asset Deleted

QR Generated
```

Events describe Storage activity.

They do not perform Storage activity.

---

# 15. Storage and Analytics

Analytics may observe Storage-related Events.

Examples include:

* upload counts
* asset generation counts

Analytics does not manage assets.

Storage does.

---

# 16. Storage Boundaries

Storage is responsible only for binary assets.

Examples that belong to Storage:

* avatar image
* background image
* generated QR image

Examples that do **not** belong to Storage:

* profile name
* bio
* button title
* AI suggestion
* analytics aggregate
* render object
* event record

These belong to the Data Platform Service.

---

# 17. Asset References

Product Domains should never depend directly on binary assets.

They should depend on structured asset references owned by the Data Platform Service.

Storage receives those references and resolves them to binary content.

This separation keeps Product Domains independent from storage implementation.

---

# 18. Temporary Assets

Some assets exist only temporarily.

Examples include:

* upload processing
* intermediate conversions
* temporary previews

Temporary assets should automatically disappear once they are no longer needed.

---

# 19. Replacement

Replacement is a Data Platform decision.

When the Data Platform commits a new canonical asset reference, the previous asset immediately becomes orphaned. Storage does not decide that an asset has been superseded — Data does.

Storage is responsible for detecting orphaned assets and executing their physical deletion. Old assets must not be retained after the Data Platform reference is updated. Storage does not archive replaced assets. Minime V1 does not support a user-accessible media history or file gallery.

Replacement never changes ownership and never creates a new Account.

---

# 20. Cleanup

Storage should remove:

* orphaned assets
* unused temporary files
* replaced assets no longer referenced
* failed uploads

Cleanup should be:

* safe
* repeatable
* idempotent

Cleanup must never remove assets that are still referenced by canonical data.

Storage identifies orphaned assets by checking for assets no longer referenced by any canonical Data Platform record. Storage does not independently determine business lifecycle — it acts on reference state owned by the Data Platform.

---

# 21. Cost Awareness

Storage directly affects operating cost.

The Storage Platform Service should always prefer:

* smaller assets
* fewer duplicates
* efficient processing
* minimal long-term storage
* automatic cleanup

Every unnecessary stored asset increases operational cost.

Storage should continuously minimize waste.

---

# 22. Technology Independence

This specification intentionally avoids choosing:

* cloud provider
* storage engine
* object storage
* filesystem
* CDN
* image library

Those are implementation decisions.

Storage Architecture must remain valid regardless of implementation technology.

---

# 23. V1 Non-Goals

The Storage Platform Service does not define:

* upload APIs
* image processing libraries
* CDN providers
* object storage vendors
* filesystem layout
* backup strategy
* disaster recovery
* database or storage replication
* server or OS management
* SSL/TLS termination
* load balancers
* CDN infrastructure management
* infrastructure monitoring
* deployment architecture

Those belong to the deployment environment, not to the product architecture.

Minime does not implement backup logic. Minime does not define a Backup Domain or a Backup Platform Service. Backups, replication, and disaster recovery are deployment concerns outside Minime's ownership.

---

# 24. V1 Storage Policy

The following policies apply specifically to Minime V1.

---

## No User Media Library

Minime V1 does not implement a user media library.

Users do not own a historical file archive.

Only the currently active asset for each purpose is retained.

---

## Current Asset Only

Storage retains only the current active asset per reference.

When a user replaces an image, the previous image must be deleted after the new reference is committed.

Historical or replaced assets must not accumulate.

---

## Canonical Image Format

Accepted upload inputs may include JPG, JPEG, PNG, HEIC, and WebP.

The permanent stored format for all user images is WebP.

Original uploaded files are temporary processing inputs. Originals must be removed after successful optimization. The architecture must not imply permanent storage of original user uploads.

---

## Canonical QR Format

The canonical stored format for QR Code assets is SVG.

PNG QR images, if needed for compatibility or download, are generated artifacts and must not be treated as the canonical QR source. PNG QR files are not retained permanently.

---

## Unsupported Media Types

Minime V1 must not permanently store:

* video
* animated GIF
* PDF
* ZIP or archive files
* arbitrary documents
* executable or script files

These are outside V1 scope unless explicitly introduced by a future architecture revision.

---

## Storage Is Not an Archive

Storage is optimized for current state only.

Default rules for all future feature evaluation:

* replace instead of accumulate
* delete instead of archive
* generate instead of store where regeneration is inexpensive
* retain only what cannot be reasonably regenerated or re-entered

Any future feature that introduces permanent files or increases retained storage must justify that retention explicitly in a domain specification.

---

## Upload Constraints

Each domain specification that uses uploaded assets defines whether upload constraints apply and the rationale for those constraints.

The API validation layer enforces domain-defined constraints before passing assets to the Storage Platform.

The Storage Platform validates that received assets are processable binary objects. Storage does not own business-level size or dimension policies.

Upload constraint values are defined by the owning domain specification — for example, profile image constraints belong to Profile Content, background image constraints belong to Appearance, image block constraints belong to the Image Block specification. Platform Architecture does not own individual domain upload limits.

---

## Capacity Planning Assumption

The V1 storage architecture should remain valid from 100,000 users to 1,000,000 users under these constraints:

* no user media library
* no permanent original uploads
* no video storage
* no arbitrary document storage
* current asset only per reference
* WebP image storage
* SVG QR storage
* generated artifacts are regenerable
* infrastructure concerns outside Minime

This is an architecture validity assumption, not a runtime performance SLA.

---

# 25. Required Follow-Up Files

This specification is the parent document for Storage.

The following files should expand specific responsibilities:

```text
platform/storage/asset.lifecycle.specification.v1.md
platform/storage/asset.processing.specification.v1.md
platform/storage/asset.delivery.specification.v1.md
platform/storage/storage.cost.optimization.specification.v1.md
```

These documents must follow the principles defined here.

---

# 26. Final Canon

The Storage Platform Service owns binary assets.

The Data Platform Service owns structured information about those assets.

Product Domains own why assets exist.

Rendering prepares how assets appear.

Public Profile delivers assets to visitors.

Storage manages the complete lifecycle of every binary asset while remaining independent from product meaning, implementation technology, and business behavior.

This separation keeps Minime simple, efficient, scalable, and inexpensive to operate.
