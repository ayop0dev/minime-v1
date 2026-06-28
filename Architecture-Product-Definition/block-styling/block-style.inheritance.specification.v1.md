# Block Style Inheritance Specification V1

## Purpose

This document defines how block styles inherit values from Themes.

The inheritance system allows blocks to automatically receive visual styles from the active Theme while supporting local customization when needed.

---

# Philosophy

Themes define visual defaults.

Blocks inherit those defaults automatically.

Rule:

```text
Theme Defines

Blocks Inherit
```

Inheritance reduces duplication and maintains visual consistency.

---

# Inheritance Goal

Without inheritance:

```text
Every Block
Requires Full Styling
```

With inheritance:

```text
Theme Defines Once

Blocks Reuse Automatically
```

This is the preferred Minime model.

---

# Inheritance Hierarchy

Block styles inherit values using a fixed hierarchy.

```text
Theme Default
        ↓
Block Override
        ↓
Resolved Style
```

The nearest valid value always wins.

---

# Theme Default Layer

The Theme provides visual defaults.

Examples:

```text
Button Radius
Button Background
Button Border
Button Shadow

Image Radius

Text Color
Text Size
```

These values are available to all compatible blocks.

---

# Block Layer

Each block may define local overrides.

Example:

```text
Button Radius
=
24px
```

This value affects only the current block.

---

# Resolved Layer

Before rendering:

```text
Theme Values
+
Block Values
```

become:

```text
Resolved Style
```

The renderer consumes only resolved values.

---

# Inherited State

A style property may remain inherited.

Example:

```text
Radius
=
Inherited
```

Result:

```text
Use Theme Radius
```

No value is stored on the block.

---

# Override State

A style property may define a local value.

Example:

```text
Radius
=
30px
```

Result:

```text
Ignore Theme Radius
Use Block Radius
```

---

# Property-Level Inheritance

Inheritance occurs per property.

Example:

```text
Theme

Radius = 16px
Shadow = Soft
Color = Black
```

Block:

```text
Radius = 24px
```

Resolved:

```text
Radius = 24px
Shadow = Soft
Color = Black
```

Only the overridden property changes.

---

# Partial Overrides

Blocks may override one property while inheriting all others.

Example:

```text
Override Background Only
```

Result:

```text
Background = Local

Everything Else
=
Inherited
```

This is fully supported.

---

# Full Overrides

Blocks may override all supported properties.

Example:

```text
Radius
Background
Border
Shadow
Color
Alignment
```

All within Theme constraints.

---

# Empty Style Object

If no overrides exist:

```text
Style
=
Empty
```

Result:

```text
100% Theme Inheritance
```

This is the default state.

---

# Block Type Inheritance

Inheritance applies to all block types.

Examples:

```text
Image
Avatar
Name
Bio
Button
Text
Divider
Social Icons
```

Each block inherits only relevant properties.

---

# Theme Change Behavior

When Theme defaults change:

```text
Theme Updated
        ↓
Inherited Values Updated
        ↓
Blocks Re-render
```

All inheriting blocks automatically reflect the new Theme.

---

# Override Protection

Theme changes never overwrite block overrides.

Example:

Theme:

```text
Radius
16px → 20px
```

Block Override:

```text
Radius = 30px
```

Result:

```text
Still 30px
```

Overrides always remain intact.

---

# Theme Switching

When a Theme changes:

```text
Old Theme
        ↓
New Theme
```

All inherited values are recalculated.

Block overrides remain unchanged.

---

# Constraint Relationship

Inherited values are assumed valid.

Override values must pass Theme constraint validation.

Flow:

```text
Override
        ↓
Validation
        ↓
Inheritance Resolution
        ↓
Rendering
```

---

# Storage Principle

Inherited values are not stored on blocks.

Only overrides are stored.

Rule:

```text
Store Overrides

Do Not Store Inheritance
```

This reduces duplication and simplifies Theme updates.

---

# Resolution Principle

Inheritance is resolved before rendering.

Rule:

```text
Theme
        ↓
Inheritance
        ↓
Resolved Style
        ↓
Renderer
```

Rendering never performs inheritance calculations.

---

# Not Supported In V1

```text
Inheritance Chains
Parent Block Inheritance
Page-Level Style Inheritance
Conditional Inheritance
Multi-Theme Inheritance
Cross-Block Inheritance
```

---

# V1 Principles

```text
Themes Define Defaults

Blocks Inherit Automatically

Overrides Replace Inherited Values

Inheritance Happens Per Property

Empty Styles Are Fully Inherited

Theme Changes Update Inherited Values

Overrides Survive Theme Changes

Only Overrides Are Stored

Inheritance Resolves Before Rendering
```
