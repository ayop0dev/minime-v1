# Minime Bio Renderer Specification V1

## Purpose

The Bio Renderer is responsible for rendering bio reference blocks.

The Bio Renderer does not own bio content.

Bio content is owned by Profile Content.

The renderer resolves the reference and returns a Render Object that carries the resolved content together with the Resolved Style produced by the Block Styling System.

---

## Block Type

Supported block type:

```text
bio
```

The Bio Renderer processes only Bio Blocks.

---

## Renderer Category

Bio Renderer is a:

```text
Reference Renderer
```

It renders profile-owned content.

It does not render block-owned content.

---

## Ownership Model

### Profile Content Owns

```text
ProfileContent.bio
```

---

### Bio Block Owns

```text
Placement
Visibility
Existence
```

The block determines whether the bio appears in the profile.

The block does not own the bio content itself.

---

## Data Source

The Bio Renderer resolves:

```text
ProfileContent.bio
```

The renderer must never read bio content from the block.

---

## Renderer Input

The renderer receives:

### Block

```text
Bio Block
```

---

### Rendering Context

Including:

```text
Profile
```

Profile Content contains the bio.

### Resolved Style

The renderer also receives the **Resolved Style** for this block, produced by the Block Styling System. The renderer attaches it to the Render Object unchanged and never reads the Theme directly.

---

## Resolution Flow

```text
Bio Block
↓
Read ProfileContent.bio
↓
Resolve Bio
↓
Create Render Object
↓
Return Result
```

---

## Successful Resolution

If a valid bio exists:

```text
ProfileContent.bio
```

the renderer returns a Render Object.

---

## Render Object

Example:

```json
{
  "id": "block_bio_1",
  "type": "bio",
  "content": {
    "value": "Founder, builder, and entrepreneur."
  },
  "resolved_style": {}
}
```

---

## Content Structure

### value

Represents the resolved profile bio.

Example:

```json
{
  "value": "Founder, builder, and entrepreneur."
}
```

---

## Required Content

Required:

```text
value
```

---

## Empty Bio

If:

```text
ProfileContent.bio = null
```

or

```text
ProfileContent.bio = empty
```

the renderer must return:

```text
null
```

No Render Object should be generated.

---

## Invalid Bio

Examples:

```text
Missing Value
Corrupted Data
Invalid Data
```

The renderer must return:

```text
null
```

---

## Content Preservation

The Bio Renderer must return the bio exactly as stored.

The renderer must not:

```text
Rewrite
Summarize
Translate
Optimize
Enhance
Generate
Reformat
```

the bio content.

---

## Line Break Preservation

The Bio Renderer should preserve stored bio formatting.

Examples:

```text
Line Breaks
Paragraph Breaks
```

must remain intact in the returned content value.

The renderer does not alter formatting.

---

## Renderer Responsibilities

The Bio Renderer may:

* Resolve profile bio references
* Validate bio availability
* Produce Render Objects

---

## Renderer Non-Responsibilities

The Bio Renderer must not:

* Create bios
* Edit bios
* Improve bios
* Rewrite bios
* Store bios
* Analyze bios

These responsibilities belong elsewhere.

---

## Presentation Independence

The Bio Renderer must not define:

```text
Font
Font Size
Color
Alignment
Spacing
Line Height
Animation
```

Examples:

```text
Centered Bio
Large Bio
Muted Bio
```

are resolved by the Block Styling System and attached to the Render Object as Resolved Style.

The renderer does not resolve or decide them.

---

## Theme Independence

The Bio Renderer must not know:

```text
Theme
Layout
Template
Visual Style
```

The renderer returns content plus the Resolved Style it is given. It never reads the Theme.

---

## Multiple Bio Blocks

V1 allows:

```text
Maximum 1 Bio Block
```

Therefore:

```text
Maximum 1 Bio Renderer Execution
```

per profile render cycle.

---

## Rendering Failure

The renderer returns:

```text
null
```

when bio content cannot be resolved safely.

The renderer must never throw public-facing errors.

---

## Public Profile Flow

```text
Bio Block
↓
Bio Renderer
↓
Render Object (content + resolved style)
↓
Rendering Engine
↓
Public Profile
```

---

## V1 Scope

Supported:

```text
Profile Bio Resolution
Render Object Creation
Null-safe Rendering
Bio Formatting Preservation
```

Not Supported:

```text
Generated Bios
AI Bios
Localized Bios
Conditional Bios
Multiple Bios
Rich Text Rendering
Markdown Rendering
```

---

## Final Rule

The Bio Renderer answers one question:

```text
What bio data should be available for presentation?
```

It does not answer:

```text
How should the Theme be resolved, or how should blocks be composed into a page?
```
