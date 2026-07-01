# Block Style Constraints Specification V1

## Purpose

This document defines the constraints applied to Block Style Overrides.

The Block Style Constraints System ensures that block-level customization remains valid, consistent, and compatible with the active Theme.

---

# Philosophy

Overrides are allowed.

Overrides are not unrestricted.

Rule:

```text
Override
Within Constraints
```

Every override must comply with the active Theme rules and the capabilities of the block type.

---

# Responsibility

The Block Style Constraints System is responsible for:

* Validating style overrides
* Enforcing Theme boundaries
* Enforcing block-type capabilities
* Preventing invalid style combinations
* Protecting rendering consistency

---

# Relationship With Theme Constraints

Theme Constraints define:

```text
What Is Allowed
```

Block Style Constraints enforce:

```text
Whether A Specific Override
Is Allowed
```

Example:

Theme:

```text
Radius Range
0px → 40px
```

Block Override:

```text
Radius = 24px
```

Result:

```text
Valid
```

---

# Validation Scope

Validation occurs on:

```text
Style Override Creation
Style Override Update
Theme Switch
Theme Configuration Change
```

Every override must remain valid.

---

# Constraint Categories

```text
Block Style Constraints
├─ Property Constraints
├─ Value Constraints
├─ Type Constraints
├─ Capability Constraints
└─ Compatibility Constraints
```

---

# Property Constraints

Only supported properties may be overridden.

Example:

Button:

```text
Radius
Background
Border
Shadow
```

Allowed.

---

Divider:

```text
Radius
```

May not be supported.

Result:

```text
Rejected
```

---

# Value Constraints

Override values must remain within Theme limits.

Example:

Theme:

```text
Radius
0px → 40px
```

Override:

```text
Radius = 60px
```

Result:

```text
Rejected
```

---

# Type Constraints

Values must match the expected type.

Example:

Radius:

```text
24px
```

Valid.

---

Radius:

```text
Blue
```

Invalid.

---

# Capability Constraints

Each block type supports a specific styling surface.

Example:

Button:

```text
Background
Radius
Shadow
Border
```

Supported.

---

Divider:

```text
Background Image
```

Not Supported.

---

Result:

```text
Rejected
```

---

# Compatibility Constraints

Some style combinations may be disallowed.

Example:

```text
Transparent Background
+
Opaque Shadow Model
```

May be incompatible.

Result:

```text
Rejected
```

Compatibility rules are defined by the styling system.

---

# Theme Change Validation

When a Theme changes:

```text
Theme A
        ↓
Theme B
```

Existing overrides must be revalidated.

---

Example:

Old Theme:

```text
Radius
0px → 40px
```

Override:

```text
30px
```

Valid.

---

New Theme:

```text
Radius
0px → 20px
```

Override:

```text
30px
```

Invalid.

---

Result:

```text
Revalidation Required
```

---

# Invalid Override Handling

If an override becomes invalid:

```text
Override
        ↓
Validation Failure
```

The system must resolve the issue before rendering.

---

Two distinct situations require two distinct strategies — they are not interchangeable options for the same event:

**At override write time** (`addBlock`/`updateBlock` submitting a new override value against the currently active Theme): **Reject Save.** The write is rejected with a validation error; the Block's stored `style_overrides` is unchanged. This is ordinary input validation, not a theme-switch concern.

**At theme-switch time** (an existing, previously-valid override becomes invalid because the newly selected Theme's constraints are narrower): **Reset To Inherited**, per affected field only. The specific override key that is now out of range is removed from `style_overrides` for that block, so the field falls back to full inheritance from the new Theme. Every other override key on the block that is still valid under the new Theme's constraints is left untouched. The theme switch itself is never blocked or rejected because of this — a user must always be able to switch themes.

**Auto-Correct** (silently substituting a different value the user did not choose, such as clamping 30px to the nearest valid bound) **is not used in V1.** It is excluded because it would make `appearance_config` diverge from anything the user actually entered, without an explicit user action — this is the same "current state remains authoritative to the user's own input" principle applied elsewhere in the architecture.

---

# Rendering Protection

Invalid overrides must never reach rendering.

Flow:

```text
Override
        ↓
Validation
        ↓
Approved Override
        ↓
Rendering
```

Only approved values are rendered.

---

# Block Capability Model

Every block type defines its styling capabilities.

Example:

Button:

```text
Background
Radius
Border
Shadow
Typography
```

---

Image:

```text
Radius
Border
Shadow
```

---

Divider:

```text
Color
Thickness
Spacing
```

---

The styling system must respect these capabilities.

---

# Override Removal

Removing an override always remains valid.

Flow:

```text
Override Removed
        ↓
Inheritance Restored
```

No validation failure can occur.

---

# Constraint Enforcement Layer

Constraint validation occurs before style resolution.

Flow:

```text
Override
        ↓
Validation
        ↓
Resolution
        ↓
Rendering
```

---

# Not Supported In V1

```text
Custom Validation Rules
User Defined Constraints
Conditional Constraints
Responsive Constraints
State-Based Constraints
Scripted Constraints
```

---

# V1 Principles

```text
Overrides Must Be Valid

Overrides Must Respect Theme Rules

Overrides Must Respect Block Capabilities

Invalid Values Are Rejected

Invalid Overrides Never Reach Rendering

Validation Happens Before Resolution

Theme Changes Trigger Revalidation

Constraint Enforcement Is Deterministic
```
