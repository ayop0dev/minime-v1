# Minime Avatar Renderer Specification V1

## Purpose

The Avatar Renderer is responsible for rendering avatar reference blocks.

The Avatar Renderer does not own avatar content.

Avatar content is owned by Profile Content.

The renderer resolves the reference and returns a Render Object that carries the resolved content together with the Resolved Style produced by the Block Styling System.

---

## Block Type

Supported block type:

```text
avatar
```

The Avatar Renderer processes only Avatar Blocks.

---

## Renderer Category

Avatar Renderer is a:

```text
Reference Renderer
```

It renders profile-owned content.

It does not render block-owned content.

---

## Ownership Model

### Profile Content Owns

```text
ProfileContent.avatar
```

---

### Avatar Block Owns

```text
Placement
Visibility
Existence
```

The block determines whether the avatar appears in the profile.

The block does not own the avatar image itself.

---

## Data Source

The Avatar Renderer resolves:

```text
ProfileContent.avatar
```

The renderer must never read avatar content from the block.

---

## Renderer Input

The renderer receives:

### Block

```text
Avatar Block
```

---

### Rendering Context

Including:

```text
Profile
```

Profile Content contains the avatar reference.

### Resolved Style

The renderer also receives the **Resolved Style** for this block, produced by the Block Styling System. The renderer attaches it to the Render Object unchanged and never reads the Theme directly.

---

## Resolution Flow

```text
Avatar Block
↓
Read ProfileContent.avatar
↓
Resolve Avatar
↓
Create Render Object
↓
Return Result
```

---

## Successful Resolution

If a valid avatar exists:

```text
ProfileContent.avatar
```

the renderer returns a Render Object.

---

## Render Object

Example:

```json
{
  "id": "block_avatar_1",
  "type": "avatar",
  "content": {
    "url": "https://cdn.minime.ai/avatar.jpg"
  },
  "resolved_style": {}
}
```

---

## Content Structure

### url

Represents the resolved avatar image URL.

Example:

```json
{
  "url": "https://cdn.minime.ai/avatar.jpg"
}
```

---

## Required Content

Required:

```text
url
```

---

## Empty Avatar

If:

```text
ProfileContent.avatar = null
```

or

```text
ProfileContent.avatar = empty
```

the renderer must return:

```text
null
```

No Render Object should be generated.

---

## Invalid Avatar

Examples:

```text
Missing URL
Corrupted Reference
Deleted Asset
Invalid Asset
```

The renderer must return:

```text
null
```

---

## Renderer Responsibilities

The Avatar Renderer may:

* Resolve avatar references
* Validate avatar availability
* Produce Render Objects

---

## Renderer Non-Responsibilities

The Avatar Renderer must not:

* Upload avatars
* Edit avatars
* Crop avatars
* Resize avatars
* Generate avatars
* Store avatars
* Replace avatars

These responsibilities belong elsewhere.

---

## Presentation Independence

The Avatar Renderer must not define:

```text
Shape
Border Radius
Size
Alignment
Position
Shadow
Border
Animation
```

Examples:

```text
Circle Avatar
Rounded Avatar
Square Avatar
```

are resolved by the Block Styling System and attached to the Render Object as Resolved Style.

The renderer does not resolve or decide them.

---

## Theme Independence

The Avatar Renderer must not know:

```text
Theme
Layout
Template
Visual Style
```

The renderer returns content plus the Resolved Style it is given. It never reads the Theme.

---

## Multiple Avatar Blocks

V1 allows:

```text
Maximum 1 Avatar Block
```

Therefore:

```text
Maximum 1 Avatar Renderer Execution
```

per profile render cycle.

---

## Rendering Failure

The renderer returns:

```text
null
```

when avatar content cannot be resolved safely.

The renderer must never throw public-facing errors.

---

## Public Profile Flow

```text
Avatar Block
↓
Avatar Renderer
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
Profile Avatar Resolution
Render Object Creation
Null-safe Rendering
```

Not Supported:

```text
Avatar Variants
Avatar Styles
Avatar Transformations
Avatar Fallback Generation
Multiple Avatars
Conditional Avatars
```

---

## Final Rule

The Avatar Renderer answers one question:

```text
What avatar data should be available for presentation?
```

It does not answer:

```text
How should the Theme be resolved, or how should blocks be composed into a page?
```
