# Layout System Specification V1

## Status

Approved

---

# Purpose

The Layout System is responsible for defining the overall structure of a profile.

Layouts control profile structure.

Layouts do not control content.

Layouts do not control block order.

Layouts do not control block visibility.

Layouts do not modify Render Objects.

---

# Position In Architecture

```text
Render Objects

+

Layout System

↓

Rendering Engine

↓

Public Profile
```

---

# Responsibilities

The Layout System is responsible for:

```text
Profile Width

Profile Container

Global Alignment

Global Spacing Rules

Profile Structure
```

---

# Not Responsible For

The Layout System is NOT responsible for:

```text
Content

Block Data

Render Objects

Block Order

Block Visibility

Block Positioning

Block Rearrangement

Theme Rules

Analytics

Tracking
```

---

# Layout Philosophy

A Layout answers:

```text
How is the profile structured?
```

A Layout never answers:

```text
Which block appears first?

Which block appears last?

What content appears?

How should content look?
```

---

# Relationship With Blocks

Blocks remain an ordered list.

Example:

```text
Avatar
↓
Name
↓
Bio
↓
Button
↓
Social Icons
```

The Layout System must preserve block order.

The Layout System must never reorder blocks.

---

# Relationship With Themes

Layouts and Themes are independent.

Layout controls:

```text
Structure
```

Theme controls:

```text
Appearance
```

Example:

```text
Layout
→ Width
→ Alignment
→ Container

Theme
→ Colors
→ Typography
→ Shadows
```

---

# Relationship With Render Objects

Layouts consume Render Objects.

Layouts never modify:

```text
id
type
content
resolved_style
```

inside Render Objects.

---

# Layout Structure Example

Example:

```json
{
  "id": "standard",
  "name": "Standard Layout",
  "container": {
    "maxWidth": "profile"
  },
  "alignment": "center"
}
```

---

# Layout Selection

Each profile references exactly one active layout.

Example:

```json
{
  "layout": "standard"
}
```

---

# Layout Switching

Changing a layout must not modify:

```text
Profile Data

Block Data

Render Objects
```

Only the profile structure changes.

---

# V1 Layout Scope

Supported:

```text
Single Active Layout

Global Profile Alignment

Global Container Rules

Global Width Rules
```

Not Supported:

```text
Per-Block Layout Rules

Grid Systems

Multi-Column Layouts

Block Rearrangement

Conditional Layouts

Nested Layouts
```

---

# Architectural Rules

Layout data must remain external to Render Objects.

Forbidden:

```text
Layout Inside Block Data

Layout Inside Render Objects

Layout Inside Content Objects
```

---

# Core Principle

Renderers define content.

Themes define appearance.

Layouts define structure.

Block order belongs to the profile.
