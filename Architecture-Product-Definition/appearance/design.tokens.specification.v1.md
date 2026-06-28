# Design Tokens Specification V1

## Status

Approved

---

# Purpose

Design Tokens are the foundational visual values used throughout the presentation system.

They provide a centralized source of truth for visual decisions.

Themes consume Design Tokens.

Layouts may consume Design Tokens.

Render Objects never contain raw Design Tokens or token references. A Render Object's resolved style carries final values that were already computed from Design Tokens by the Theme and Block Styling Systems.

---

# Position In Architecture

```text
Design Tokens
↓
Theme (defaults)
↓
Block Styling (resolution)
↓
Resolved Style
↓
Render Object
↓
Rendering Engine
↓
Public Profile
```

Design Tokens feed the Theme. The Theme's defaults are resolved into final per-block styles by the Block Styling System before rendering. Tokens are never applied at the Rendering Engine.

---

# Responsibilities

Design Tokens are responsible for defining:

```text
Colors

Typography

Spacing

Border Radius

Shadows

Sizing

Motion Values
```

---

# Not Responsible For

Design Tokens are NOT responsible for:

```text
Content

Profile Data

Block Data

Render Objects

Block Order

Profile Structure

Theme Selection

Layout Selection
```

---

# Design Token Philosophy

Design Tokens answer:

```text
What visual values are available?
```

They do not answer:

```text
How should the profile look?

How should blocks be arranged?

What content should appear?
```

---

# Token Categories

## Colors

Examples:

```text
Primary

Secondary

Background

Surface

Text

Border

Accent
```

---

## Typography

Examples:

```text
Font Family

Font Size

Font Weight

Line Height

Letter Spacing
```

---

## Spacing

Examples:

```text
XS

SM

MD

LG

XL
```

---

## Radius

Examples:

```text
None

Small

Medium

Large

Full
```

---

## Shadows

Examples:

```text
Small

Medium

Large
```

---

## Sizing

Examples:

```text
Avatar Sizes

Container Widths

Maximum Content Widths
```

---

## Motion

Examples:

```text
Transition Duration

Animation Duration

Interaction Timing
```

---

# Example Structure

```json
{
  "colors": {},
  "typography": {},
  "spacing": {},
  "radius": {},
  "shadows": {},
  "sizing": {},
  "motion": {}
}
```

---

# Relationship With Themes

Themes consume tokens.

Themes may combine tokens.

Themes may reference tokens.

Themes must not duplicate token definitions.

Example:

```text
Design Tokens
↓
Theme
↓
Block Styling
↓
Resolved Style
```

---

# Relationship With Layouts

Layouts may consume:

```text
Spacing

Sizing
```

Tokens.

Layouts must not define their own token system.

---

# Relationship With Render Objects

Render Objects must not contain raw Design Tokens or token references.

Forbidden inside Render Objects:

```text
Token Names

Token References

Unresolved Token Values
```

A Render Object's `resolved_style` carries final style values (colors, typography, spacing, shadows, radius) that were already computed from Design Tokens by the Theme and Block Styling Systems. These are final, resolved values — not tokens to be looked up at render time.

---

# Single Source Of Truth

All visual values must originate from Design Tokens.

Themes and Layouts consume tokens.

They do not redefine them.

---

# V1 Scope

Supported:

```text
Centralized Token System

Theme Consumption

Layout Consumption

Global Visual Consistency
```

Not Supported:

```text
User-Generated Tokens

Runtime Token Creation

Per-Profile Token Editing

Token Overrides
```

---

# Architectural Rules

Design Tokens must remain independent from:

```text
Content

Blocks

Render Objects

Profile Data
```

Design Tokens are presentation infrastructure only.

---

# Core Principle

Design Tokens define values.

Themes define appearance.

Layouts define structure.

Renderers define content.
