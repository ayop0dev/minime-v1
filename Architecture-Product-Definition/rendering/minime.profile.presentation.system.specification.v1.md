# Minime Profile Presentation System Specification V1

## Status

Approved

---

# Purpose

The Profile Presentation System is responsible for composing render objects into a final visual profile experience.

It sits above the Rendering layer, as a layer under the Rendering Engine.

It does not own profile content.

It does not own block content.

It does not render block data.

It does not apply the Theme, resolve styles, or define the rendering architecture. Render objects arrive already carrying their Resolved Style; presentation only composes them into a page.

Its responsibility is page-level composition only.

---

# Position In Architecture

```text
Account
↓
Account Management
↓
Profile Content
↓
Blocks
↓
Rendering
↓
Profile Presentation
↓
Public Profile
```

---

# Responsibilities

The Profile Presentation System is responsible for:

```text
Layout Selection
Layout Application

Visual Composition (of already-styled Render Objects)

Presentation Rules

Page Assembly

Public Profile Output
```

The active Theme reference is recorded at the profile level, but Theme resolution and application happen upstream in the Theme Application System and the Block Styling System. Presentation never applies the Theme.

---

# Not Responsible For

The Profile Presentation System is NOT responsible for:

```text
Authentication

Username Management

Connected Accounts

Social Accounts Setup

Profile Content Management

Block Content Management

Block Validation

Rendering

Analytics

Tracking

Storage
```

---

# Inputs

The Profile Presentation System receives:

```text
Profile Configuration

Layout Configuration

Render Objects (content + resolved style)
```

Example:

```json
[
  {
    "id": "block_001",
    "type": "avatar",
    "content": {},
    "resolved_style": {}
  },
  {
    "id": "block_002",
    "type": "name",
    "content": {},
    "resolved_style": {}
  }
]
```

---

# Outputs

The Profile Presentation System produces:

```text
Visual Profile Experience
```

The exact UI implementation is outside the scope of this specification.

---

# Relationship With Rendering

Profile Presentation consumes Render Objects.

Profile Presentation never creates Render Objects.

Profile Presentation never modifies Render Objects.

Profile Presentation never performs data normalization.

Profile Presentation never reads raw block data.

---

# Relationship With Themes

Themes define:

```text
Colors
Typography
Spacing
Radius
Shadows
Visual Rules
```

Themes do not own content.

Themes do not own block data.

Themes do not modify render objects.

---

# Relationship With Layouts

Layouts define:

```text
Block Positioning

Block Width Rules

Vertical Rhythm

Container Structure

Alignment Rules
```

Layouts do not modify content.

Layouts do not modify render objects.

---

# Profile Assembly Flow

```text
Profile Request
↓
Load Profile Configuration
↓
Load Layout
↓
Load Render Objects (already styled)
↓
Compose Render Objects
↓
Assemble Visual Profile
↓
Return Public Profile
```

---

# Architectural Rule

Presentation is a consumer.

Rendering is a producer.

Profile Presentation must never move responsibilities back into the rendering layer.

---

# V1 Scope

Supported:

```text
Single Profile

Single Page

Ordered Block List

Composition Of Already-Styled Render Objects

Layout System
```

Not Supported:

```text
Subpages

Nested Pages

Multi-page Navigation

Conditional Rendering

Dynamic Layout Switching

Block Groups
```
