# appearance.state.specification.v1.md

# Minime Appearance State Specification V1

**Version:** 1.0
**Status:** Approved
**Domain:** Appearance
**Architecture Status:** Frozen

---

# 1. Purpose

Appearance State defines the complete visual configuration of a Profile.

It represents the user's selected visual preferences after all customization has been applied.

Appearance State exists solely to persist visual configuration and provide Rendering with the information required to produce the Public Profile.

It does not define rendering behavior, layouts, runtime decisions, or content.

---

# 2. Scope

Appearance State is responsible for persisting visual configuration for a single Profile.

It serves as the permanent visual state consumed by the Rendering system.

Appearance State belongs exclusively to the Appearance domain.

---

# 3. Ownership

Appearance State is owned by the Appearance domain.

Only the Appearance domain may create, modify, validate, or persist Appearance State.

Other domains may consume Appearance State but never modify it directly.

---

# 4. Responsibilities

Appearance State is responsible for:

* Persisting visual configuration
* Persisting selected Theme information
* Persisting user customization
* Providing Rendering with visual configuration
* Maintaining visual consistency
* Preserving user appearance preferences

---

# 5. Non-Responsibilities

Appearance State is never responsible for:

* Rendering
* Layout generation
* Block rendering
* Content management
* Analytics
* Account management
* Publishing
* Runtime calculations
* Theme definitions
* Theme catalogs
* Animation execution
* Rendering optimization

---

# 6. Appearance State Structure

Appearance State represents the final visual configuration for a Profile.

Conceptually it contains:

* Selected Theme
* Theme Variant (if applicable)
* User customizations
* Visual overrides
* Appearance preferences

The exact internal representation is implementation-specific.

---

# 7. Persistence Model

Appearance State is persisted as part of the Profile Content record.

It is not an independent business entity.

It exists only within the context of its owning Profile.

There is exactly one Appearance State for each Profile.

---

# 8. Lifecycle

Appearance State is created when a Profile is initialized.

It evolves as users modify their visual preferences.

Updates replace previous visual configuration while preserving ownership.

Appearance State remains valid throughout the lifetime of the Profile.

It is deleted only when its owning Profile is permanently removed.

---

# 9. Update Rules

Appearance State may only be updated through the Appearance domain.

Every update must produce a valid Appearance State.

Partial updates are permitted provided the resulting state remains valid.

Invalid updates must never be persisted.

---

# 10. Validation Rules

Before persistence, Appearance State shall satisfy all Appearance validation rules.

Validation includes:

* Supported Theme selection
* Valid customization values
* Constraint compliance
* Visual compatibility
* Version compatibility

No invalid Appearance State may be persisted.

---

# 11. Theme Relationship

Appearance State references a Theme selected from the Theme Library.

Theme Library provides reusable visual definitions.

Appearance State records which Theme has been selected together with any permitted user customization.

Appearance State never owns Theme definitions.

Theme definitions remain global resources managed independently of individual Profiles.

---

# 12. Rendering Relationship

Rendering consumes Appearance State as read-only input.

Appearance State provides visual configuration.

Rendering produces visual output.

Rendering never modifies Appearance State.

Appearance does not participate in rendering execution.

Conceptually:

Theme Library

↓

Appearance State

↓

Rendering

↓

Public Profile

---

# 13. Stored Data

Appearance State may contain only persistent visual configuration, including:

* Theme selection
* Appearance preferences
* User-approved visual customization
* Appearance configuration values
* Visual option selections

Only user-visible visual configuration is persisted.

---

# 14. Non-Persisted Data

Appearance State never stores:

* Rendered HTML
* CSS output
* Rendering artifacts
* Layout trees
* Runtime calculations
* Cached rendering results
* Analytics
* Performance metrics
* Derived values
* Computed styles
* Rendering instructions
* Temporary editor state
* Preview state
* Session state

These values belong to runtime systems rather than persistent storage.

---

# 15. Invariants

The following architectural rules are permanent.

## Profile Ownership

Each Profile owns exactly one Appearance State.

---

## Domain Ownership

Appearance State belongs exclusively to the Appearance domain.

---

## Theme Separation

Theme Library is global.

Appearance State is profile-specific.

---

## Rendering Separation

Rendering consumes Appearance State.

Rendering never owns Appearance State.

---

## Persistence Separation

Only persistent visual configuration is stored.

Runtime artifacts are never persisted.

---

## Content Separation

Appearance State contains no Profile content.

Profile content belongs to the Profile Content domain.

---

## Runtime Separation

Appearance State contains no runtime execution data.

---

## Derived Data Separation

Computed values are never persisted.

Only source configuration is stored.

---

## Single Source of Truth

Appearance State is the single authoritative source for persistent visual configuration.

---

# 16. Future Evolution

Future versions may introduce additional appearance capabilities without changing the architectural role of Appearance State.

New visual properties shall extend the existing model while preserving:

* Domain ownership
* Persistence boundaries
* Rendering separation
* Theme separation
* Architectural invariants

The responsibilities defined by this specification are considered stable for Minime V1.
