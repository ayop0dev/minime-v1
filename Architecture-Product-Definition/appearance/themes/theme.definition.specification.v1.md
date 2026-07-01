# Theme Definition Specification V1

## Status

**V1 Status:** The Theme Definition catalog structure described in this document (Metadata, Background, Colors, Typography, Spacing, Radius, Borders, Shadows, Animations, Block Defaults, Constraints) is real, approved V1 behavior — every Theme in the V1 catalog has these fields as fixed platform-provided defaults, consumed during rendering.

**What is NOT V1:** User customization of any of these values (the "Theme Identity Rule" example below) is **V2 Scope**. In V1, `appearance_config.customizations` is always `{}` — see `theme.customization.specification.v1.md` for the full V2 customization design and the governing decision (`ARCHITECTURE_PR_APPROVAL_DECISIONS.md` — APD-011). This document's references to "customization" describe the V2 target design unless stated otherwise.

---

## Context

A Theme Definition is a catalog asset within the Theme Library.

The Theme Library is a subsystem of the Appearance system.

A Theme Definition describes a reusable visual preset that profiles may select.

It is platform-managed and immutable.

The Appearance system stores the selected Theme reference (V1) and, in a future version, customization values (V2 Scope) as the Appearance State.

---

## Purpose

This document defines the structure and content of a Theme Definition.

A Theme Definition provides defaults, constraints, and identity.

It feeds into the Appearance State during resolution.

---

## Theme Definition Is A Catalog Asset

```text
Theme Definition   →   Theme Catalog   →   Appearance system
```

A Theme Definition lives in the Theme Catalog.

It is not profile-owned.

User customization is stored in the Appearance State, not inside the Theme Definition.

Rule:

```text
Theme Definitions Are Immutable Catalog Assets
Appearance State Is Mutable Profile-Level Data
```

---

## Theme Definition Structure

```text
Theme Definition
├─ Metadata
├─ Background
├─ Colors
├─ Typography
├─ Spacing
├─ Radius
├─ Borders
├─ Shadows
├─ Animations
├─ Block Defaults
└─ Constraints
```

---

## Metadata

Identifies the Theme Definition uniquely.

Fields:

```text
theme_id
theme_key
theme_name
theme_version
```

---

## Background

Defines profile background appearance defaults.

Fields:

```text
background_type
background_configuration
```

Supported background types are defined by Theme Constraints.

---

## Colors

Defines the color system defaults.

Fields:

```text
primary_color
secondary_color
text_color
surface_color
accent_color
```

---

## Typography

Defines typography defaults.

Fields:

```text
heading_font
body_font
heading_size
body_size
font_weight
line_height
```

---

## Spacing

Defines spacing scale defaults.

Fields:

```text
spacing_scale
block_gap
section_gap
```

---

## Radius

Defines corner radius defaults.

Fields:

```text
radius_xs
radius_sm
radius_md
radius_lg
radius_xl
```

---

## Borders

Defines border defaults.

Fields:

```text
border_width
border_style
border_color
```

---

## Shadows

Defines shadow defaults.

Fields:

```text
shadow_style
shadow_strength
```

---

## Animations

Defines animation defaults.

Fields:

```text
animation_preset
animation_speed
```

---

## Block Defaults

Defines default visual values for each block type.

The Theme Definition provides these defaults.

They are consumed during Appearance State resolution.

Structure:

```text
Block Type → Default Visual Values
```

Examples:

```text
Button
Image
Name
Bio
Text
Divider
Social Icons
```

Example — Button Defaults:

```text
Button
├─ Background Color
├─ Text Color
├─ Radius
├─ Border
└─ Shadow
```

---

## Constraints

Defines customization limits that protect Theme visual identity.

Constraint types:

```text
Allowed Radius Range
Allowed Font Choices
Allowed Color Rules
Allowed Background Types
Allowed Animation Presets
```

Constraint details are defined in: theme.constraints.specification.v1.md

---

## Theme Identity Rule (V2 Scope)

Theme identity remains unchanged when users customize their Appearance State.

Example:

```text
Profile references: Theme = Minimal
User changes: primary color, radius, typography
Result: Profile still references Theme = Minimal (with customized values in Appearance State)
```

Customization does not create a new Theme Definition.

**V1 behavior:** there is no customization to apply in V1. Selecting a Theme sets `selected_theme_id` only; the Theme's own defaults are used unmodified. The identity-survival rule above becomes directly applicable once V2 customization is implemented.

---

## Downstream Rendering

Theme Definitions provide defaults that flow into the Appearance State.

The Renderer consumes the resolved Appearance State.

Rule:

```text
Theme Definition (defaults)
        ↓
Appearance State (V1: defaults only — V2: resolved with customization)
        ↓
Renderer
```

The Theme Definition never communicates directly with the Renderer.

---

## Block-Level Style Overrides — Resolved, Separate From This Document (V1)

An earlier draft of this document flagged block instance visual overrides as pending a separate V1 Appearance review before being introduced or expanded.

That review is complete. Block-level style overrides are approved, in-scope V1 behavior, governed entirely by the `block-styling/` specification set (`block-style.model.specification.v1.md`, `block-style.constraints.specification.v1.md`, `block-style.inheritance.specification.v1.md`, `block-style.override.specification.v1.md`, `block-style.resolution.specification.v1.md`) and `implementation/07-validation-rules.md`.

Block-level style overrides are a distinct mechanism from the theme-wide customization described in `theme.customization.specification.v1.md` (V2 Scope). A block override changes one property on one block instance; theme-wide customization (V2) would change a value once for every block that inherits it. The existing Block Defaults structure in this document (the Theme's own per-block-type defaults) is used as-is in V1 and is unaffected by either mechanism.

---

## Theme Definition Ownership

Theme Definition owns:

```text
Background Defaults
Color Defaults
Typography Defaults
Spacing Defaults
Radius Defaults
Border Defaults
Shadow Defaults
Animation Defaults
Block Type Defaults
Constraints
```

Theme Definition does not own:

```text
Profile Content
Blocks
Links
Analytics
Account Data
Appearance State
```

---

## V1 Principles

```text
Theme Definition Is A Catalog Asset

Theme Definition Is Platform-Managed And Immutable

Theme Definition Provides Defaults Only

Appearance State Stores Theme Selection (selected_theme_id) In V1

Theme Definition Does Not Own Profile Content

Renderer Consumes Appearance State, Not Theme Definition Directly

Block-Level Style Overrides Are A Separate, Already-Approved V1 Capability — See block-styling/
```

## V2 Principles (Target Design, Not Yet Implemented)

```text
Appearance State Also Stores Theme-Wide Customization Values

Theme Identity Survives Customization
```
