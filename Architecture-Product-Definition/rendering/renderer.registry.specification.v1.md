# Minime Renderer Registry Specification V1

## Purpose

The Renderer Registry is responsible for mapping block types to their corresponding renderers.

It provides a single source of truth for renderer resolution within the Rendering domain.

---

## Core Principle

A block renderer must never be selected through scattered conditional logic.

Avoid:

```text
if block.type == avatar
if block.type == name
if block.type == button
...
```

The system should resolve renderers through a centralized registry.

---

## Responsibility

The Renderer Registry is responsible for:

* Mapping block types to renderers
* Resolving the correct renderer for a block
* Providing a predictable rendering path
* Preventing renderer ambiguity

---

## Non-Responsibilities

The Renderer Registry does NOT:

* Render blocks
* Own content
* Modify content
* Validate content
* Store content
* Manage themes
* Manage layouts
* Manage presentation rules

Its only responsibility is renderer resolution.

---

## Rendering Flow

```text
Block
↓
Renderer Registry
↓
Renderer Resolution
↓
Renderer Execution (Block Data + Resolved Style)
↓
Render Object (content + resolved style)
```

The registry resolves only which renderer runs. The resolved renderer is executed with Block Data and the Resolved Style produced by the Block Styling System, and returns a Render Object that carries both.

---

## Registry Structure

Each supported block type must have exactly one renderer.

Example:

```text
avatar
↓
AvatarRenderer

name
↓
NameRenderer

bio
↓
BioRenderer

image
↓
ImageRenderer

social_icons
↓
SocialIconsRenderer

button
↓
ButtonRenderer

divider
↓
DividerRenderer

title
↓
TitleRenderer

textbox
↓
TextboxRenderer
```

---

## V1 Registry

### Reference Block Renderers

```text
avatar
→ AvatarRenderer

name
→ NameRenderer

bio
→ BioRenderer
```

---

### Content Block Renderers

```text
image
→ ImageRenderer

button
→ ButtonRenderer

divider
→ DividerRenderer

title
→ TitleRenderer

textbox
→ TextboxRenderer
```

---

### Collection Reference Block Renderers

```text
social_icons
→ SocialIconsRenderer
```

---

## One Renderer Per Block Type

A block type may only have one active renderer.

Valid:

```text
button
→ ButtonRenderer
```

Invalid:

```text
button
→ ButtonRendererV1

button
→ ButtonRendererV2

button
→ AlternativeButtonRenderer
```

The registry must always resolve a single renderer.

---

## Renderer Resolution

When a block enters the rendering pipeline:

```text
Button Block
↓
Registry Lookup
↓
ButtonRenderer
↓
Render Object
```

The renderer is selected using the block type.

No other property participates in renderer selection.

---

## Resolution Key

Renderer resolution is based exclusively on:

```text
block.type
```

Examples:

```text
avatar
name
bio
button
image
social_icons
```

Renderer selection must not depend on:

```text
theme
layout
user plan
profile settings
presentation mode
device type
```

---

## Unknown Block Types

If a block type is not registered:

```text
Renderer Not Found
```

The rendering pipeline must safely return:

```text
null
```

The system must not crash.

---

## Registry Stability

Renderer mappings should remain stable.

Changing a renderer mapping is considered a system-level change.

Example:

```text
button
→ ButtonRenderer
```

should remain consistent across the system.

---

## Renderer Independence

The registry resolves renderers individually.

A renderer must never require:

```text
another renderer
another block
another registry entry
```

Each renderer operates independently.

---

## Registry Ownership

The Renderer Registry belongs to:

```text
Rendering
```

It does not belong to:

```text
Profile Content

Appearance

Public Profile

Layouts
```

---

## Extending The Registry

New block types may be added in future versions.

Example:

```text
video
→ VideoRenderer

gallery
→ GalleryRenderer
```

Adding a new block type requires:

1. Block Definition
2. Renderer Definition
3. Registry Entry

---

## V1 Scope

Supported renderer mappings:

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

Only.

---

## Not Supported In V1

The Renderer Registry does not support:

```text
Plugin Renderers
Custom Renderers
Third-Party Renderers
Marketplace Renderers
Dynamic Registration
Runtime Registration
Renderer Overrides
Theme-Based Renderer Selection
Layout-Based Renderer Selection
User-Defined Renderers
```

---

## Final Rule

The Renderer Registry answers one question:

```text
Which renderer should process this block?
```

It does not answer:

```text
How should this block be rendered?
```

That responsibility belongs to the renderer itself.
