# Rendering Engine Specification V1

## Status

Approved

---

# Purpose

The Rendering Engine is responsible for composing a visual profile from Render Objects, which already carry their Resolved Style, using the selected Layout.

It is the execution layer of the Profile Presentation System.

It does not render data.

It does not transform content.

It does not build Render Objects.

It does not apply the Theme, resolve styles, or process inheritance and overrides. Style is already resolved upstream by the Block Styling System and carried inside each Render Object.

---

# Position In Architecture

```text
Theme
↓
Block Styling
↓
Resolved Style
↓
Block Renderers
↓
Render Objects (content + resolved style)
↓
Rendering Engine
↓
Public Profile UI
```

---

# Responsibilities

The Rendering Engine is responsible for:

```text
Loading Render Objects (already carrying resolved style)

Loading Layout

Ordering Blocks

Composing Already-Styled Render Objects

Assembling Final Profile Structure

Producing Final Visual Output
```

---

# Not Responsible For

The Rendering Engine is NOT responsible for:

```text
Block Data Access

Content Normalization

Data Transformation

Block Validation

Theme Application

Style Resolution

Inheritance

Override Processing

Theme Creation

Layout Creation

Analytics

Tracking

Storage

Authentication
```

---

# Inputs

The Rendering Engine receives:

```text
Render Objects (content + resolved style)

Layout Configuration
```

Example:

```json
{
  "layout": "standard",
  "blocks": [
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
}
```

---

# Render Object Contract

The Rendering Engine only consumes Render Objects.

Expected structure:

```json
{
  "id": "...",
  "type": "...",
  "content": {},
  "resolved_style": {}
}
```

Every Render Object already carries its Resolved Style. The Rendering Engine consumes that resolved style as-is and must never require:

```text
Raw Block Data

Database Records

Raw Theme Data

Theme Configuration

Layout Information

HTML

CSS
```

inside a Render Object.

---

# Rendering Flow

```text
Load Layout
↓
Load Render Objects (already styled)
↓
Apply Layout Rules
↓
Compose Render Objects
↓
Assemble Profile
↓
Return Public UI
```

---

# Block Processing

The Rendering Engine processes blocks as an ordered list.

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

The engine preserves profile-defined order.

The engine does not reorder blocks automatically.

---

# Style Is Pre-Resolved

The Rendering Engine does not apply the Theme.

Each Render Object already carries its Resolved Style — the final values produced by the Block Styling System.

```text
Colors

Typography

Spacing

Radius

Shadows

Visual Effects
```

are read from the Render Object's resolved style. The Rendering Engine never recomputes them from the Theme, inheritance, or overrides.

---

# Layout Application

The Rendering Engine applies:

```text
Alignment

Container Width

Vertical Spacing

Block Positioning

Profile Structure
```

according to the selected layout.

Layout data remains external to Render Objects.

---

# Error Handling

If a Render Object type is unknown:

```text
Skip Object
Log Error
Continue Rendering
```

The profile should remain renderable whenever possible.

---

# Architectural Rule

The Rendering Engine is a consumer of Render Objects.

The Rendering Engine must never perform responsibilities that belong to Block Renderers.

Forbidden:

```text
Reading Raw Profile Data

Reading Raw Block Data

Normalizing Content

Transforming Content

Creating Render Objects

Applying The Theme

Resolving Styles

Processing Inheritance Or Overrides
```

---

# Cross-Cutting Capability Contracts

The Rendering Engine is the primary consumer of both Cross-Cutting Product Capabilities: SEO and Integrations.

Neither is a Product Domain. Neither is a Platform Service.

Both feed into Rendering without owning rendering behavior.

---

## SEO Contract

The SEO capability defines what metadata must exist for every public page.

The Rendering Engine is responsible for emitting that metadata into the final HTML output.

Authority boundary:

```text
SEO owns:   metadata generation rules
Rendering owns: HTML emission
```

The Rendering Engine must never:

* invent SEO metadata independently of SEO definitions
* override robots directives, canonical URLs, or Open Graph values
* decide indexing behavior

The SEO capability must never:

* directly emit HTML
* modify Render Objects
* own rendering behavior

Dependency direction:

```text
Rendering depends on SEO.
SEO does not depend on Rendering.
```

---

## Integrations Contract

The Integrations capability defines which external providers are eligible to load on a public page.

The Rendering Engine is responsible for activating approved providers during public page generation.

Authority boundary:

```text
Integrations owns:  provider eligibility determination
Rendering owns:     provider loading execution
```

The Rendering Engine must never:

* independently decide which providers exist or are eligible
* bypass Integrations eligibility rules
* expose the platform to unapproved external code

The Integrations capability must never:

* directly execute provider loading
* modify Render Objects
* own rendering behavior
* interrupt rendering if a provider fails

Provider failures are isolated. A provider failure must never prevent the profile from rendering.

Dependency direction:

```text
Rendering depends on Integrations.
Integrations does not depend on Rendering.
```

---

## Circular Dependency Prohibition

Neither SEO nor Integrations may depend on Rendering.

Rendering may depend on both SEO and Integrations.

No circular dependency is permitted in this three-way relationship.

---

# V1 Scope

Supported:

```text
Single Page Profiles

Ordered Block Rendering

Composition Of Already-Styled Render Objects

Layout Application

Render Object Consumption

SEO Metadata Emission (defined by SEO capability)

Integration Provider Loading (approved by Integrations capability)
```

Not Supported:

```text
Conditional Rendering

A/B Rendering

Dynamic Layout Switching

Runtime Content Transformation

Multi-page Rendering
```
