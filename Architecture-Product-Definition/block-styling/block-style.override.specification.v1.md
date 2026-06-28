# Block Style Override Specification V1

## Purpose

This document defines the Block Style Override system used by Minime.

Overrides allow individual blocks to customize their visual appearance without modifying the active Theme.

Overrides affect only the targeted block instance.

---

# Philosophy

Themes provide defaults.

Overrides provide exceptions.

Rule:

```text
Theme
=
Default

Override
=
Exception
```

Overrides exist to allow visual flexibility while preserving Theme consistency.

---

# Definition

A Block Style Override is a locally defined style value that replaces an inherited Theme value.

Example:

```text
Theme Radius
=
16px

Block Radius
=
24px
```

Result:

```text
24px
```

for that block only.

---

# Scope

Overrides apply only to:

```text
Single Block Instance
```

They never affect:

```text
Other Blocks
Theme Defaults
Profile Configuration
```

---

# Override Ownership

Overrides belong to blocks.

Rule:

```text
Theme
Owns Defaults

Block
Owns Overrides
```

Overrides are stored inside the block.

---

# Override Structure

A block may contain:

```text
Block
├─ Content
├─ Settings
└─ Style Overrides
```

The Style Overrides section is optional.

---

# Supported Override Types

Overrides may exist for any style property supported by the block.

Examples:

```text
Color
Background
Radius
Border
Shadow
Typography
Alignment
Spacing
```

Supported properties depend on block type.

---

# Property-Level Overrides

Overrides operate per property.

Example:

Theme:

```text
Radius = 16px
Shadow = Soft
Color = Black
```

Block:

```text
Radius = 24px
```

Result:

```text
Radius = 24px
Shadow = Soft
Color = Black
```

Only the overridden property changes.

---

# Partial Overrides

Blocks may override a single property.

Example:

```text
Background Only
```

Everything else remains inherited.

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

Within Theme constraints.

---

# Empty Overrides

Blocks may contain no overrides.

Example:

```text
Style Overrides
=
Empty
```

Result:

```text
Fully Inherit Theme
```

This is the default state.

---

# Override Precedence

Override values always take priority over inherited Theme values.

Priority order (highest to lowest):

```text
Block Override
        ↓
Theme Configuration
        ↓
Theme Defaults
```

Higher levels replace lower levels.

Ownership note:

Theme Defaults, Theme Configuration, and Theme Constraints are owned by the Appearance domain.
Block Styling receives them as inputs.
Block Styling never owns or modifies them.

---

# Override Persistence

Overrides remain attached to the block.

Example:

```text
Block Radius = 24px
```

remains:

```text
24px
```

until explicitly changed or removed.

---

# Theme Change Behavior

Theme changes do not remove overrides.

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
30px
```

Overrides remain unchanged.

---

# Theme Switch Behavior

When Themes change:

```text
Old Theme
        ↓
New Theme
```

Overrides remain attached to blocks.

Only inherited values are recalculated.

---

# Override Removal

Users may remove an override.

Flow:

```text
Override Removed
        ↓
Property Returns To Inheritance
```

Example:

```text
Radius Override Deleted
```

Result:

```text
Use Theme Radius
```

---

# Override Validation

All overrides must pass Theme constraints.

Flow:

```text
Override
        ↓
Validation
        ↓
Store
```

Invalid overrides are rejected.

---

# Constraint Enforcement

Overrides may never bypass Theme rules.

Example:

Theme Constraint:

```text
Radius
0px → 40px
```

Override:

```text
60px
```

Result:

```text
Rejected
```

---

# Storage Principle

Only explicit overrides are stored.

Inherited values are never stored.

Example:

Stored:

```text
Radius = 24px
```

Not Stored:

```text
Radius = Inherited
```

---

# Resolution Relationship

Overrides participate in style resolution.

Flow:

```text
Theme Defaults
        ↓
Theme Configuration
        ↓
Block Overrides
        ↓
Resolved Style
```

The Renderer consumes only the resolved result.

---

# Override Independence

Overrides affect appearance only.

Overrides may not modify:

```text
Content
Layout
Block Order
Theme Definition
Profile Structure
```

---

# Not Supported In V1

```text
Conditional Overrides
Responsive Overrides
Scheduled Overrides
State-Based Overrides
Cross-Block Overrides
Page-Level Overrides
Shared Override Presets
```

---

# V1 Principles

```text
Overrides Belong To Blocks

Overrides Replace Inherited Values

Overrides Affect One Block Only

Overrides Survive Theme Changes

Overrides Survive Theme Switching

Overrides Must Respect Constraints

Empty Overrides Are Valid

Removing Overrides Restores Inheritance

Only Explicit Overrides Are Stored
```
