# Minime Asset Delivery Specification V1

**Status:** Canonical
**Version:** V1
**Platform Service:** Storage
**Parent:** `platform/storage/storage.architecture.specification.v1.md`
**Repository State:** Canonical Repository State V1.0

---

# 1. Purpose

This document defines how binary assets leave the Storage Platform Service and become available to the rest of Minime.

Asset Delivery is responsible for making assets consumable.

It is not responsible for:

* storing assets
* processing assets
* rendering assets
* serving HTTP requests
* choosing delivery technology

Those responsibilities belong elsewhere.

Asset Delivery defines the architectural contract between Storage and every consumer of binary assets.

---

# 2. Core Principle

Storage owns binary assets.

Consumers never own binary assets.

Consumers request assets.

Storage delivers them.

Consumers should never know:

* where assets are stored
* how assets are stored
* how assets were processed
* how assets are optimized

Delivery hides storage implementation.

---

# 3. Delivery Philosophy

Delivery exists to expose assets without exposing Storage.

Every consumer should interact with assets through stable references.

Never through storage implementation details.

Changing Storage implementation must not affect Product Domains.

---

# 4. Consumers

The Storage Platform Service may deliver assets to:

* Rendering
* Public Profile
* AI (where permitted)
* future internal services

Product Domains do not consume binary assets directly.

They consume structured references owned by the Data Platform Service.

---

# 5. Delivery Flow

The canonical flow is:

```text
Product Domain

↓

Structured Asset Reference

↓

Data Platform Service

↓

Storage Platform Service

↓

Binary Asset

↓

Consumer
```

Every layer owns exactly one responsibility.

---

# 6. Stable References

Consumers should receive stable asset references.

They must never depend on:

* filesystem paths
* bucket names
* cloud providers
* internal storage layout

Changing internal Storage organization must not affect consumers.

---

# 7. Delivery Is Read-Only

Delivery exposes assets.

It does not modify them.

Consumers must not change assets through Delivery.

Asset creation, replacement, and deletion belong to the Storage lifecycle.

---

# 8. Rendering Relationship

Rendering consumes delivered assets.

Rendering decides presentation.

Storage does not decide presentation.

Storage delivers binary content.

Rendering composes the visitor experience.

---

# 9. Public Profile Relationship

Public Profile serves assets already prepared by Storage.

Public Profile should never:

* optimize assets
* replace assets
* process uploads
* manage lifecycle

Public Profile delivers.

Storage prepares.

---

# 10. AI Relationship

AI may consume assets only when Product Domain rules allow it.

Examples include:

* avatar analysis
* profile image suggestions
* future visual recommendations

AI never owns delivered assets.

AI must never replace user assets automatically.

---

# 11. Data Relationship

Data owns structured references.

Storage resolves those references.

Delivery never replaces Data ownership.

Delivery simply transforms an approved reference into consumable binary content.

---

# 12. Events Relationship

Meaningful delivery operations may emit Events.

Examples include:

* Asset Delivered
* Asset Missing
* Delivery Failed

Events describe delivery activity.

They do not perform delivery.

---

# 13. Missing Assets

Consumers should never assume an asset always exists.

Delivery must safely handle:

* deleted assets
* missing assets
* invalid references
* expired assets

Failure should remain predictable.

Broken assets must never corrupt Product Domain behavior.

---

# 14. Asset Availability

Delivery should expose only assets that are valid for consumption.

Assets that are:

* temporary
* failed
* orphaned
* deleted

must not appear as normal delivered assets.

---

# 15. Consistency

The same asset reference should always resolve to the same intended asset while it remains valid.

Consumers should experience consistent behavior regardless of Storage implementation.

---

# 16. Security Boundary

Asset Delivery does not decide authorization.

Authorization belongs to Product Domains and application workflows.

Delivery assumes the request has already been approved.

Its responsibility begins after authorization.

---

# 17. Technology Independence

Asset Delivery intentionally avoids defining:

* HTTP endpoints
* CDN behavior
* object storage APIs
* signed URLs
* filesystem access
* cloud vendor features

Those belong to implementation.

The architectural contract remains unchanged regardless of technology.

---

# 18. V1 Non-Goals

This specification does not define:

* image streaming
* adaptive media
* video delivery
* edge caching
* CDN configuration
* delivery protocols
* transport optimization

Those may evolve during implementation without affecting architecture.

---

# 19. Final Canon

Asset Delivery exists to make binary assets available without exposing Storage implementation.

Storage owns binary assets.

Data owns structured references.

Rendering consumes assets.

Public Profile delivers visitor experiences.

AI consumes approved assets when allowed.

Every consumer interacts with stable references rather than storage details.

This separation keeps Storage replaceable, Product Domains independent, and Minime simple to maintain regardless of future infrastructure choices.
