# Minime Render Object Specification V1

## Purpose

This document defines the normalized object returned by every block renderer in Minime V1.

The Render Object exists to keep the rendering layer:

* Lightweight
* Simple
* Predictable
* Cacheable
* Extensible

---

## Core Decision

In Minime V1:

```text
Renderer = Data Transformer
```

A renderer does not build UI.

A renderer receives Block Data together with the Resolved Style produced by the Block Styling System, and returns a Render Object that carries both the normalized content and the resolved style.

The renderer never resolves the Theme, inheritance, or overrides. Those are already resolved upstream by the Block Styling System.

---

## Render Object Definition

A Render Object is the normalized data structure returned by a block renderer.

It represents the final renderable data for one block: its content plus its Resolved Style.

It carries:

* Normalized content
* Resolved Style (final, already-computed style values)

It does not represent:

* UI components
* HTML
* CSS
* Raw Theme data or Theme configuration
* Unresolved inheritance or overrides
* Layout rules
* Visual composition

The Resolved Style it carries is final. The Render Object must contain enough information for the Rendering Engine to compose the final output without re-resolving styles or touching the Theme.

---

## Standard Render Object Shape

Every renderer must return the same base structure:

```json
{
  "id": "block_xxx",
  "type": "button",
  "content": {},
  "resolved_style": {}
}
```

The per-type examples later in this document show the `content` shape only and omit `resolved_style` for brevity. Every Render Object carries a `resolved_style` object as shown in this base structure.

---

## Required Fields

### id

The original block instance ID.

Used for:

* Stable rendering
* Tracking
* Analytics
* Caching
* Debugging
* Future hydration

Renderers must not generate a new ID.

---

### type

The block type being rendered.

Examples:

```text
avatar
name
bio
image
social_icons
button
divider
title
textbox
```

The `type` value must match the original block type.

Renderers must not change the block type.

---

### content

The normalized renderable content for the block.

The shape of `content` depends on the block type.

Examples:

```json
{
  "label": "Visit Website",
  "url": "https://example.com"
}
```

```json
{
  "value": "About Me"
}
```

```json
{
  "accounts": []
}
```

---

### resolved_style

The final, already-computed style object for the block, produced by the Block Styling System and passed into the renderer.

The renderer attaches this object to the Render Object unchanged. It does not compute, resolve, or modify it.

`resolved_style` contains only final values, for example:

```json
{
  "colors": {},
  "typography": {},
  "background": {},
  "radius": {},
  "border": {},
  "shadow": {},
  "alignment": {},
  "spacing": {}
}
```

---

## Forbidden Fields

A Render Object must not contain:

```text
theme
theme_config
theme_constraints
inheritance
overrides
layout
component
props
html
css
className
container
database
storage
editor
admin
```

These belong to other layers.

Note: final style values (colors, typography, radius, shadow, spacing, alignment, etc.) are permitted only inside the `resolved_style` object, because they have already been resolved by the Block Styling System. Raw Theme data, Theme configuration, unresolved inheritance, and overrides remain forbidden.

---

## Renderer Output Rule

Renderers return data only: normalized content plus the Resolved Style they were given.

They do not return:

* Markup
* Components
* Framework-specific structures
* Raw Theme data or styling instructions to be resolved
* Layout instructions

The resolved style attached to the Render Object is final data, not a styling instruction to be computed downstream.

---

## Example Render Objects

### Avatar

```json
{
  "id": "block_avatar_1",
  "type": "avatar",
  "content": {
    "url": "https://example.com/avatar.jpg"
  }
}
```

---

### Name

```json
{
  "id": "block_name_1",
  "type": "name",
  "content": {
    "value": "Ahmed Hassan"
  }
}
```

---

### Bio

```json
{
  "id": "block_bio_1",
  "type": "bio",
  "content": {
    "value": "Founder, designer, and builder."
  }
}
```

---

### Image

```json
{
  "id": "block_image_1",
  "type": "image",
  "content": {
    "url": "https://example.com/image.jpg",
    "alt": "Profile image"
  }
}
```

---

### Button

```json
{
  "id": "block_button_1",
  "type": "button",
  "content": {
    "label": "Visit Website",
    "url": "https://example.com"
  }
}
```

---

### Divider

```json
{
  "id": "block_divider_1",
  "type": "divider",
  "content": {}
}
```

---

### Title

```json
{
  "id": "block_title_1",
  "type": "title",
  "content": {
    "value": "Projects"
  }
}
```

---

### Textbox

```json
{
  "id": "block_textbox_1",
  "type": "textbox",
  "content": {
    "value": "I build simple digital products for creators and founders."
  }
}
```

---

### Social Icons

```json
{
  "id": "block_social_icons_1",
  "type": "social_icons",
  "content": {
    "accounts": [
      {
        "id": "connected_account_1",
        "platform": "instagram",
        "username": "ahmed",
        "url": "https://instagram.com/ahmed"
      }
    ]
  }
}
```

---

## Content Normalization

Renderers may normalize content to make it safe and predictable for presentation.

Allowed normalization includes:

* Removing empty values
* Returning empty strings instead of null when appropriate
* Returning empty arrays instead of null for collections
* Resolving references
* Filtering invisible collection items

Renderers must not rewrite user content for style or meaning.

---

## Reference Blocks

Reference blocks resolve profile-owned content.

Used by:

```text
avatar
name
bio
```

Example:

```text
Avatar Block
→ ProfileContent.avatar
→ Render Object
```

The block owns placement only.

The profile owns the actual content.

---

## Content Blocks

Content blocks render their own stored content.

Used by:

```text
image
button
divider
title
textbox
```

Example:

```text
Button Block
→ Button.label + Button.url
→ Render Object
```

---

## Collection Reference Blocks

Collection reference blocks resolve external collections.

Used by:

```text
social_icons
```

Example:

```text
Social Icons Block
→ Connected Accounts
→ Apply selection
→ Apply ordering
→ Apply visibility
→ Render Object
```

The renderer may include only the connected accounts selected by the block.

---

## Rendering Failure

If a block cannot be rendered safely, the renderer should return `null`.

Examples:

* Missing required reference
* Deleted connected account
* Empty required content
* Invalid block type

The renderer must not throw user-facing errors during public profile rendering.

---

## Empty Content Rules

If a block has no meaningful renderable content:

```text
Return null
```

Examples:

* Empty name
* Empty bio
* Button without label
* Button without URL
* Image without URL
* Title without text
* Textbox without text
* Social Icons with no visible accounts

Exception:

```text
Divider
```

Divider may return a valid Render Object with empty content because it is visual separation only.

---

## Rendering Order

Render Objects must preserve the block order received from Profile Content.

Renderers do not decide ordering.

The rendering pipeline may skip blocks that return `null`.

---

## Rendering Engine Relationship

The Rendering Engine receives Render Objects that already carry their Resolved Style.

The Rendering Engine is responsible for composing Render Objects into the final profile output.

The Rendering Engine owns:

* Composition
* Assembly
* Page-level output construction
* Layout (structure) application

The Rendering Engine does not own:

* Theme application
* Style resolution
* Inheritance
* Override processing

It assembles already-styled Render Objects. It must never calculate styles from Themes or overrides, because the style is already resolved and carried by each Render Object.

---

## Public Profile Relationship

Public Profile does not read raw blocks directly.

Public Profile receives the output composed by the Rendering Engine from Render Objects.

Flow:

```text
Theme
↓
Block Styling
↓
Resolved Style
↓
Renderers (Block Data + Resolved Style)
↓
Render Objects (content + resolved style)
↓
Rendering Engine
↓
Public Profile
```

---

## V1 Scope

Render Objects in V1 support:

* Flat block rendering
* Lightweight normalized data
* Reference resolution
* Collection resolution
* Resolved style delivery (per-block, already computed)
* Public profile rendering

Render Objects in V1 do not support:

* Nested children
* Parent-child relationships
* Multi-page structures
* UI component mapping
* Raw Theme configuration
* Layout configuration
* Renderer plugins
* Custom renderer registration
* Style resolution (resolution happens upstream in Block Styling; the Render Object only carries the result)

---

## Final Rule

A Render Object must answer one question only:

```text
What content and resolved style should this block be presented with?
```

It must not answer:

```text
How should the Theme be resolved?
How should this block's style be inherited or overridden?
How should blocks be composed into a page?
```

Theme resolution and style resolution happen upstream in Block Styling. Composition happens downstream in the Rendering Engine.
