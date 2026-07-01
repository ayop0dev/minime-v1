# Theme Customization Specification

## Status

**V1 Status:** V2 Scope (not implemented in V1). See "V1 Scope Notice" below.
**Document Status:** Approved — describes intended future (V2) behavior only.

---

## V1 Scope Notice

**This entire document describes a V2 capability. None of it is implemented in V1.**

Minime V1 supports **Theme Selection only**: an account selects one Theme from the Theme Catalog (`theme.catalog.specification.v1.md`, `theme.selection.specification.v1.md`). V1 does not support editing any Theme value.

Concretely, in V1:

- `ProfileContent.appearance_config.customizations` is always an empty object `{}`.
- No customization field described in this document (background, colors, typography, spacing, radius, borders, shadows, animations) is defined, validated, or accepted by any V1 API endpoint.
- The Design editor (`design.editor.specification.v1.md`) exposes Theme selection only in V1. It does not expose color pickers, typography controls, spacing controls, radius controls, border controls, shadow controls, or animation controls.
- `PATCH /api/v1/profile/appearance` accepts a `customizations` field only for forward compatibility; the field must always be `{}` in V1 and any non-empty value is rejected at the validation layer.

This document remains the canonical design for the future customization engine so that V2 implementation can proceed directly from it without redesign. Every section below is written in the present tense as a *description of the target capability*, not as a statement of current V1 behavior. Every capability described below is **V2 Scope** unless explicitly marked otherwise.

This resolves the contradiction identified between this document and `implementation/03-canonical-data-model.md` (which defines `customizations` as always `{}` in V1). The canonical decision is recorded in `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` — APD-011.

---

## Context

Theme customization is the mechanism by which users modify visual values within a selected Theme.

Customization values are stored in the Appearance State, not in the Theme Definition.

This is a subsystem of the Appearance system.

Users access Theme customization through the Design editor.

---

## Purpose

This document defines all user-customizable Theme values planned for Minime, for introduction in a version after V1.

Theme customization allows users to adjust the visual appearance of their profile while remaining within Theme constraints.

---

## Philosophy

```text
Theme
=
Preset
+
Customization
```

Customization changes values within the Appearance State.

Customization does not create a new Theme Definition.

The profile continues referencing the same Theme.

---

## Customization Scope (V2 Scope)

Theme customization applies to the entire profile.

```text
Main Page
Sub Pages (if applicable)
All Blocks
```

There is no per-page customization planned.

---

## Customization Categories (V2 Scope)

```text
Theme Customization
├─ Background
├─ Colors
├─ Typography
├─ Spacing
├─ Radius
├─ Borders
├─ Shadows
└─ Animations
```

None of these categories are user-editable in V1. `appearance_config.customizations` is always `{}` in V1.

---

## Background Customization (V2 Scope)

Users may configure:

```text
Background Type
Background Settings
```

Supported types are defined by Theme Constraints.

Examples:

```text
Solid Color
Gradient
Image
```

---

## Color Customization (V2 Scope)

Users may configure:

```text
Primary Color
Secondary Color
Accent Color
Text Color
Surface Color
```

Available color controls depend on Theme Constraints.

---

## Typography Customization (V2 Scope)

Users may configure:

```text
Heading Font
Body Font
Heading Size
Body Size
Line Height
Font Weight
```

Available font options are defined by Theme Constraints.

---

## Spacing Customization (V2 Scope)

Users may configure:

```text
Spacing Scale
Block Gap
Section Gap
```

Spacing values affect all inheriting blocks.

---

## Radius Customization (V2 Scope)

Users may configure:

```text
Corner Radius Scale
```

Examples:

```text
Small Radius
Medium Radius
Large Radius
```

Changing radius values updates all blocks that inherit Theme defaults.

---

## Border Customization (V2 Scope)

Users may configure:

```text
Border Width
Border Style
Border Color
```

Available options depend on Theme Constraints.

---

## Shadow Customization (V2 Scope)

Users may configure:

```text
Shadow Style
Shadow Strength
```

Changes apply globally across the profile.

---

## Animation Customization (V2 Scope)

Users may configure:

```text
Animation Preset
Animation Speed
```

Animation availability depends on Theme Constraints.

---

## Global Customization Behavior (V2 Scope)

Changing a Theme value updates all elements that inherit Theme defaults.

Example:

```text
Button Radius
16px → 24px
```

All buttons inheriting Theme defaults automatically update.

---

## Theme Switching (V2 Scope)

When a Theme changes:

```text
Previous Customization Values
        ↓
Cleared
        ↓
New Theme Defaults Applied
```

The profile uses only one Theme customization set at a time.

In V1, since no customization exists, switching Themes only changes `selected_theme_id`; there are no customization values to clear.

---

## Customization Persistence

Theme customization is stored in the Appearance State.

Rule:

```text
One Profile
=
One Appearance State
=
One Theme Customization Set
```

Customization applies to the entire profile.

**V1 behavior:** `appearance_config.customizations` is always an empty object `{}`. No customization set exists to persist in V1. See `implementation/03-canonical-data-model.md` — ProfileContent.

---

## Block-Level Style Overrides — Resolved, Separate From This Document (V1)

An earlier draft of this document flagged per-block-type default customization (button/image/text/social-icon defaults) as pending a separate V1 Appearance review before being introduced.

That review is complete. Block-level style overrides are approved, in-scope V1 behavior, and are **not** part of the theme-wide customization engine described in the rest of this document. Block-level overrides are a narrower, closed-catalog mechanism (a fixed, per-block-type set of overridable keys such as a button's `radius` or a name block's `text_color`) governed entirely by:

- `block-styling/block-style.model.specification.v1.md` — the closed property catalog per block type
- `block-styling/block-style.constraints.specification.v1.md` — value ranges and the Reject-Save / Reset-To-Inherited resolution policy
- `block-styling/block-style.inheritance.specification.v1.md`, `block-style.override.specification.v1.md`, `block-style.resolution.specification.v1.md`
- `implementation/07-validation-rules.md` — `style_overrides` validation

Do not confuse the two: block-level style overrides (V1, real, implemented) let a user override a handful of specific properties on one specific block instance. Theme-wide customization (this document, V2) would let a user change a value once and have it apply to every block on the profile that inherits from the Theme. Neither implies the other.

---

## Relationship With Constraints

Theme Customization (V2 Scope) defines:

```text
What Users Can Change
```

Theme Constraints define:

```text
How Far Users Can Change It
```

Constraint rules are defined in: `theme.constraints.specification.v1.md`. Note that `theme.constraints.specification.v1.md` governs both this document's V2 theme-wide customization and the already-approved V1 block-level style overrides — the two constraint layers are described separately there.

---

## Not Supported In V1

```text
All Theme Customization described in this document (Background, Colors, Typography, Spacing,
Radius, Borders, Shadows, Animations) — V2 Scope, see "V1 Scope Notice" above
Per Page Theme Customization
Per Sub Page Theme Customization
Multiple Active Theme Customization Sets
Conditional Customization Rules
Scheduled Customization Changes
```

---

## V1 Principles (Actual V1 Behavior)

```text
V1 Supports Theme Selection Only — One Profile References One Theme From The Catalog

appearance_config.customizations Is Always {} In V1

No Theme-Wide Customization Field Is User-Editable In V1

Theme Identity Is Fixed By Selection, Not Modified By Customization

Block-Level Style Overrides Are A Separate, Already-Approved V1 Capability — See block-styling/
```

---

## V2 Principles (Target Design, Not Yet Implemented)

```text
Theme Customization Is Stored In Appearance State

Customization Is Profile-Wide

Customization Does Not Create New Theme Definitions

Theme Identity Survives Customization

Theme Constraints Define Customization Limits

One Profile = One Active Theme Customization Set
```
