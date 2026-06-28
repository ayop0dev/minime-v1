# Theme Customization Specification V1

## Context

Theme customization is the mechanism by which users modify visual values within a selected Theme.

Customization values are stored in the Appearance State, not in the Theme Definition.

This is a subsystem of the Appearance Domain.

Users access Theme customization through the Design editor.

---

## Purpose

This document defines all user-customizable Theme values available in Minime V1.

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

## Customization Scope

Theme customization applies to the entire profile.

```text
Main Page
Sub Pages (if applicable)
All Blocks
```

There is no per-page customization in V1.

---

## Customization Categories

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

---

## Background Customization

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

## Color Customization

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

## Typography Customization

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

## Spacing Customization

Users may configure:

```text
Spacing Scale
Block Gap
Section Gap
```

Spacing values affect all inheriting blocks.

---

## Radius Customization

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

## Border Customization

Users may configure:

```text
Border Width
Border Style
Border Color
```

Available options depend on Theme Constraints.

---

## Shadow Customization

Users may configure:

```text
Shadow Style
Shadow Strength
```

Changes apply globally across the profile.

---

## Animation Customization

Users may configure:

```text
Animation Preset
Animation Speed
```

Animation availability depends on Theme Constraints.

---

## Global Customization Behavior

Changing a Theme value updates all elements that inherit Theme defaults.

Example:

```text
Button Radius
16px → 24px
```

All buttons inheriting Theme defaults automatically update.

---

## Theme Switching

When a Theme changes:

```text
Previous Customization Values
        ↓
Cleared
        ↓
New Theme Defaults Applied
```

The profile uses only one Theme customization set at a time.

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

---

## V1 Flag — Block Default Customization

Some earlier design work described per-block-type default customization:

```text
Button Defaults (background, text color, radius, border, shadow)
Image Defaults (radius, border, shadow)
Text Defaults (alignment, color, size)
Social Icon Defaults (color, size, shape)
```

This area describes block-level styling that crosses into block-level customization controls.

This requires a separate V1 Appearance review before being introduced or expanded.

Do not build block-type default customization UI in V1 without completing that review.

This flag does not prevent Theme Definitions from providing block type defaults for rendering purposes.

---

## Relationship With Constraints

Theme Customization defines:

```text
What Users Can Change
```

Theme Constraints define:

```text
How Far Users Can Change It
```

Constraint rules are defined in: theme.constraints.specification.v1.md

---

## Not Supported In V1

```text
Per Page Theme Customization
Per Sub Page Theme Customization
Multiple Active Theme Customization Sets
Conditional Customization Rules
Scheduled Customization Changes
```

---

## V1 Principles

```text
Theme Customization Is Stored In Appearance State

Customization Is Profile-Wide

Customization Does Not Create New Theme Definitions

Theme Identity Survives Customization

Theme Constraints Define Customization Limits

One Profile = One Active Theme Customization Set

Block-Level Customization UI Requires Separate V1 Approval
```
