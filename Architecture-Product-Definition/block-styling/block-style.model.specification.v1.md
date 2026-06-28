# Block Style Model Specification V1

## Purpose

This document defines the Block Style data model used by the Minime Block Styling System.

The Block Style Model represents the visual appearance of an individual block instance.

It stores only styling information.

It never stores content.

---

# Philosophy

A Block Style describes:

```text
How A Block Looks
```

A Block Style does not describe:

```text
What A Block Contains
```

Content and styling are separate concerns.

---

# Style Ownership

Each block may own a style object.

Structure:

```text
Block
├─ Content
├─ Settings
└─ Style
```

The Style object belongs to the Block Styling System.

---

# Style Object

A Block Style is a collection of visual properties.

Structure:

```text
Style
├─ Colors
├─ Typography
├─ Background
├─ Radius
├─ Border
├─ Shadow
├─ Alignment
└─ Spacing
```

Not all properties apply to all block types.

---

# Style Categories

## Colors

Defines visual colors.

Examples:

```text
Text Color
Background Color
Border Color
Icon Color
```

---

## Typography

Defines text appearance.

Examples:

```text
Font Family
Font Size
Font Weight
Line Height
Text Alignment
```

Only applicable to text-based blocks.

---

## Background

Defines block background appearance.

Examples:

```text
Solid Color
Gradient
Transparent
```

Supported options depend on block type.

---

## Radius

Defines corner appearance.

Examples:

```text
0px
8px
16px
24px
```

Applicable to blocks supporting rounded corners.

---

## Border

Defines border appearance.

Examples:

```text
Border Width
Border Style
Border Color
```

---

## Shadow

Defines shadow appearance.

Examples:

```text
None
Soft
Medium
Strong
```

---

## Alignment

Defines content positioning.

Examples:

```text
Left
Center
Right
```

Supported options depend on block type.

---

## Spacing

Defines spacing values.

Examples:

```text
Padding
Margin
Internal Gap
```

---

# Style Inheritance State

Each style property exists in one of two states.

## Inherited

The value comes from the active Theme.

Example:

```text
Radius
=
Inherited
```

---

## Overridden

The value is defined directly on the block.

Example:

```text
Radius
=
24px
```

---

# Style Resolution State

Before rendering:

```text
Inherited Values
+
Override Values
```

produce:

```text
Resolved Style
```

The Renderer consumes only resolved values.

---

# Block Type Independence

The Style Model is generic.

The same structure is used by:

```text
Button
Image
Avatar
Name
Bio
Text
Divider
Social Icons
```

Each block type uses only relevant properties.

---

# Empty Style Object

Blocks may have no overrides.

Example:

```text
Style
=
Empty
```

Result:

```text
Use Theme Defaults
```

This is a valid state.

---

# Partial Style Object

Blocks may override only selected properties.

Example:

```text
Radius
=
24px
```

Everything else continues inheriting Theme values.

---

# Complete Style Object

Blocks may override all supported properties.

Example:

```text
Radius
Background
Border
Shadow
Alignment
```

Only within Theme constraints.

---

# Constraint Relationship

All style values must satisfy:

```text
Theme Constraints
```

Invalid values are rejected before rendering.

---

# Storage Principle

Block Style data belongs to the block.

Rule:

```text
Theme
Owns Defaults

Block
Owns Overrides
```

---

# Rendering Principle

The Renderer never reads:

```text
Theme Defaults
```

or

```text
Raw Overrides
```

directly.

The Renderer consumes:

```text
Resolved Style
```

only.

---

# V1 Principles

```text
Block Styles Store Appearance Only

Block Styles Never Store Content

Style Objects Are Optional

Empty Styles Are Valid

Blocks Own Style Overrides

Themes Own Style Defaults

Resolved Styles Drive Rendering

All Styles Respect Theme Constraints
```
