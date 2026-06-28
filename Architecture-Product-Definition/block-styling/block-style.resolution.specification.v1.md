# Block Style Resolution Specification V1

## Purpose

This document defines the Block Style Resolution process used by Minime.

Style Resolution combines Theme defaults, Theme configuration, block overrides, and constraint validation into a single resolved style object.

The resolved style object is the only styling object consumed by the Renderer.

---

# Philosophy

Themes are not rendered directly.

Overrides are not rendered directly.

The Renderer requires a fully resolved style object.

Rule:

```text
Theme
+
Overrides
+
Validation
=
Resolved Style
```

---

# Responsibility

The Style Resolution System is responsible for:

* Resolving inherited values
* Applying overrides
* Validating style values
* Producing final style objects
* Supplying render-ready styles

The Style Resolution System does not perform rendering.

---

# Resolution Goal

Transform:

```text
Theme Defaults
Theme Configuration
Block Overrides
```

into:

```text
Resolved Style
```

for a specific block instance.

---

# Resolution Inputs

The Style Resolution System receives:

```text
Theme Defaults
Theme Configuration
Block Type
Block Overrides
Theme Constraints
Block Constraints
```

These inputs are required for resolution.

Ownership note:

Theme Defaults, Theme Configuration, and Theme Constraints are provided by the Appearance area.
The Style Resolution System receives them as external inputs.
It does not own or produce them.

---

# Resolution Output

The system produces:

```text
Resolved Style
```

Structure:

```text
Resolved Style
├─ Colors
├─ Typography
├─ Background
├─ Radius
├─ Border
├─ Shadow
├─ Alignment
└─ Spacing
```

All values must be final.

No inheritance remains unresolved.

---

# Resolution Pipeline

Resolution follows a fixed sequence.

```text
Theme Defaults
        ↓
Theme Configuration
        ↓
Apply Constraints
        ↓
Apply Block Overrides
        ↓
Validate Overrides
        ↓
Resolved Style
```

The sequence is deterministic.

---

# Step 1 — Load Theme Defaults

The system loads the active Theme defaults.

Example:

```text
Button Radius = 16px
Button Shadow = Soft
Button Color = Black
```

These become the initial style state.

---

# Step 2 — Apply Theme Configuration

Profile-level Theme customization is applied.

Example:

Theme Default:

```text
Radius = 16px
```

Theme Configuration:

```text
Radius = 24px
```

Result:

```text
Radius = 24px
```

The configured value replaces the default value.

---

# Step 3 — Validate Theme Values

Theme values are validated against Theme constraints.

Example:

Constraint:

```text
Radius
0px → 40px
```

Value:

```text
24px
```

Result:

```text
Valid
```

Only valid values continue.

---

# Step 4 — Apply Block Overrides

Block-level overrides are applied.

Example:

Resolved Theme:

```text
Radius = 24px
```

Block Override:

```text
Radius = 32px
```

Result:

```text
Radius = 32px
```

Overrides replace inherited values.

---

# Step 5 — Validate Overrides

Overrides are validated against:

```text
Theme Constraints
Block Constraints
```

Both validations must pass.

---

# Step 6 — Generate Resolved Style

The final style object is generated.

Example:

```text
Resolved Style

Radius = 32px
Shadow = Soft
Color = Black
Border = None
```

The object contains only final values.

---

# Property Resolution

Resolution occurs independently for each property.

Example:

Theme:

```text
Radius = 16px
Shadow = Soft
Color = Black
```

Override:

```text
Radius = 24px
```

Result:

```text
Radius = 24px
Shadow = Soft
Color = Black
```

Only affected properties change.

---

# Empty Override Resolution

If no override exists:

```text
Style Overrides
=
Empty
```

Result:

```text
Use Theme Values
```

The block fully inherits the Theme.

---

# Partial Override Resolution

If only one property is overridden:

```text
Background
```

Result:

```text
Background = Override
Everything Else = Inherited
```

Partial overrides are fully supported.

---

# Full Override Resolution

If all supported properties are overridden:

```text
Radius
Background
Border
Shadow
Color
Alignment
```

Result:

```text
Use Override Values
```

for those properties.

---

# Theme Change Resolution

When Theme values change:

```text
Theme Updated
        ↓
Recalculate Resolution
        ↓
Generate New Resolved Style
```

Affected blocks receive updated styles automatically.

---

# Theme Switch Resolution

When the active Theme changes:

```text
Theme A
        ↓
Theme B
```

All blocks must be re-resolved.

Existing overrides remain attached.

---

# Override Removal Resolution

When an override is removed:

```text
Override Deleted
        ↓
Property Returns To Inheritance
        ↓
Re-resolve
```

The inherited Theme value becomes active.

---

# Resolution Caching

The platform may cache resolved styles.

Example:

```text
Resolved Style Cache
```

to reduce repeated calculations.

Caching behavior is implementation-specific.

---

# Renderer Contract

The Renderer consumes only:

```text
Resolved Style
```

The Renderer must never consume:

```text
Theme Defaults
Theme Configuration
Block Overrides
```

directly.

---

# Failure Handling

If resolution fails:

```text
Use Last Valid Resolved Style
```

or

```text
Use Valid Theme Defaults
```

depending on system policy.

Invalid style states must never reach rendering.

---

# Deterministic Resolution

Given identical inputs:

```text
Theme
Configuration
Overrides
Constraints
```

the system must always generate:

```text
The Same Resolved Style
```

Rule:

```text
Resolution Must Be Deterministic
```

---

# Performance Principle

Resolution occurs before rendering.

Rule:

```text
Resolve Once
Render Many
```

The Renderer should never perform style calculations.

---

# Not Supported In V1

```text
Runtime Style Computation
Conditional Resolution
Responsive Resolution
State-Based Resolution
Cross-Block Resolution
Multi-Theme Resolution
Scripted Resolution
```

---

# V1 Principles

```text
Resolved Style Is The Single Source Of Truth

Renderer Consumes Resolved Styles Only

Themes Are Resolved Before Rendering

Overrides Are Resolved Before Rendering

Validation Happens Before Resolution Completes

Resolution Is Deterministic

Resolution Is Repeatable

Resolution Produces Final Values Only

Resolve Once, Render Many
```
