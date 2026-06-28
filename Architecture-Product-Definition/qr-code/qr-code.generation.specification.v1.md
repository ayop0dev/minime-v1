# qr-code.generation.specification.v1.md

# Minime QR Code Generation Specification V1

**Version:** 1.0
**Status:** Approved
**Domain:** QR Code

---

# 1. Purpose

This specification defines the canonical generation model for QR Codes within Minime.

It establishes when QR Codes are generated, what they represent, and the architectural rules governing their creation.

The specification is implementation-independent and does not prescribe any QR generation library or encoding algorithm.

---

# 2. Scope

The QR Code Generation process is responsible for:

* Creating QR representations
* Associating QR Codes with Profiles
* Ensuring canonical destination integrity
* Producing reusable QR assets

It is not responsible for lifecycle management, rendering, analytics, or profile routing.

---

# 3. Generation Trigger

QR Code generation becomes eligible only after:

* A Profile exists.
* A canonical Public Profile URL has been established.

A QR Code shall never be generated for incomplete, temporary, or unestablished profile URLs.

---

# 4. Generation Target

The generated QR Code always represents the canonical Public Profile URL.

Conceptually:

```text
Profile

↓

Canonical Public Profile URL

↓

QR Generation

↓

QR Code
```

The generation process never targets:

* Preview URLs
* Editor URLs
* Temporary URLs
* Internal system endpoints

---

# 5. Canonical Identity

The QR Code represents the logical identity of the Public Profile.

Its purpose is to provide an alternative access mechanism to the same destination.

The QR Code does not define the destination.

The Public Profile remains the canonical source.

---

# 6. Generation Rules

Generation shall follow these rules:

* Exactly one active QR Code per Profile.
* Every QR Code targets one canonical Public Profile.
* Every generated QR Code is reusable.
* Generation is deterministic for the same canonical destination.
* QR generation never alters Profile data.

---

# 7. Regeneration Rules

A QR Code may be regenerated when:

* The QR asset must be recreated.
* The storage representation changes.
* Internal implementation changes.
* The canonical Public Profile URL changes.

Regeneration preserves ownership and logical identity.

---

# 8. Relationship with Public Profile

The Public Profile provides the canonical destination.

QR generation depends on the existence of the Public Profile.

The Public Profile never depends on QR generation.

Dependency is one-directional.

---

# 9. Relationship with Rendering

Rendering produces the Public Profile.

QR generation consumes the resulting canonical destination.

Rendering and QR generation remain independent systems.

Neither system owns the other.

---

# 10. Relationship with Analytics

QR generation performs no analytics.

Scanning a QR Code may later participate in analytics workflows implemented by other domains.

Analytics remain outside the scope of QR generation.

---

# 11. Validation

Before generation, the system shall verify:

* A valid Profile exists.
* A canonical Public Profile URL exists.
* The destination is valid.
* The Profile is eligible for QR generation.

Generation shall fail gracefully if these conditions are not satisfied.

---

# 12. Generated Asset

The generated QR representation shall:

* Be reusable.
* Represent a single canonical destination.
* Remain independent from presentation.
* Be suitable for digital and printed use.

The canonical stored format for QR assets is SVG. SVG provides a resolution-independent representation suitable for both digital display and print use cases.

PNG QR images, if needed for compatibility or download, are generated artifacts. They must not be treated as the canonical QR source and must not be stored permanently.

The internal storage mechanism and storage path are implementation-specific.

---

# 13. Invariants

The following architectural rules are permanent.

## One QR Per Profile

A Profile owns at most one active QR Code.

---

## Canonical Destination

Every generated QR Code references the canonical Public Profile URL.

---

## Immutable Ownership

Generation never changes Profile ownership.

---

## No Business Logic

QR generation contains no business rules.

---

## No Rendering

QR generation never renders the Public Profile.

---

## No Routing

QR generation never defines URL routing.

---

## No Analytics

QR generation never records analytics events.

---

## Deterministic Output

Generating a QR Code for the same canonical destination shall always represent the same logical destination, regardless of implementation.

---

# 14. Future Evolution

Future versions may introduce additional generation capabilities, including:

* Multiple export formats
* Branded QR designs
* Theme-aware QR styling
* High-resolution assets
* Print templates
* Vector asset generation
* Batch generation

These enhancements shall preserve:

* One QR Code per Profile
* Canonical destination integrity
* Rendering independence
* Domain ownership
* Implementation independence

The generation model defined by this specification is the canonical QR generation model for Minime V1.
