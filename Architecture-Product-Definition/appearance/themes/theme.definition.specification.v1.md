# Theme Definition Specification V1

## Context

A Theme Definition is a catalog asset within the Theme Library.

The Theme Library is a subsystem of the Appearance Domain.

A Theme Definition describes a reusable visual preset that profiles may select.

It is platform-managed and immutable.

The Appearance Domain stores the selected Theme reference and customization values as the Appearance State.

---

## Purpose

This document defines the structure and content of a Theme Definition.

A Theme Definition provides defaults, constraints, and identity.

It feeds into the Appearance State during resolution.

---

## Theme Definition Is A Catalog Asset

```text
Theme Definition   →   Theme Catalog   →   Appearance Domain
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

## Theme Identity Rule

Theme identity remains unchanged when users customize their Appearance State.

Example:

```text
Profile references: Theme = Minimal
User changes: primary color, radius, typography
Result: Profile still references Theme = Minimal (with customized values in Appearance State)
```

Customization does not create a new Theme Definition.

---

## Downstream Rendering

Theme Definitions provide defaults that flow into the Appearance State.

The Renderer consumes the resolved Appearance State.

Rule:

```text
Theme Definition (defaults)
        ↓
Appearance State (resolved with customization)
        ↓
Renderer
```

The Theme Definition never communicates directly with the Renderer.

---

## V1 Flag — Block-Level Override Model

The current model includes a concept of block instance visual overrides — values stored on individual blocks that override Theme defaults.

This area requires a separate V1 Appearance review before any expansion.

Do not introduce block-level visual override controls in V1 without completing that review.

This flag does not prevent the existing Block Defaults structure from being used as-is.

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

Appearance State Stores Selection And Customization

Theme Identity Survives Customization

Theme Definition Does Not Own Profile Content

Renderer Consumes Appearance State, Not Theme Definition Directly
```
