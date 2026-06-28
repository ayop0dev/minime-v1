# Minime Image Renderer Specification V1

## Purpose

The Image Renderer is responsible for rendering Image Blocks.

The Image Renderer renders content owned by the Image Block itself.

The renderer consumes the block content together with the Resolved Style produced by the Block Styling System, and returns a Render Object that carries both the content and the resolved style.

---

## Block Type

Supported block type:

```text
image
```

The Image Renderer processes only Image Blocks.

---

## Renderer Category

Image Renderer is a:

```text
Content Renderer
```

It renders block-owned content.

It does not resolve profile references.

It does not resolve connected account references.

---

## Ownership Model

### Image Block Owns

```text
image
alt_text
```

The Image Block is the single source of truth for its content.

---

## Data Source

The renderer reads content directly from:

```text
Image Block
```

The renderer must not read image content from:

```text
Profile
Connected Accounts
Other Blocks
```

---

## Renderer Input

The renderer receives:

### Block

```text
Image Block
```

Containing:

```text
image
alt_text
```

### Resolved Style

The renderer also receives the **Resolved Style** for this block, produced by the Block Styling System. The renderer attaches it to the Render Object unchanged and never reads the Theme directly.

---

## Resolution Flow

```text
Image Block
↓
Read Image Content
↓
Validate Content
↓
Create Render Object
↓
Return Result
```

---

## Successful Resolution

If a valid image exists:

```text
image
```

the renderer returns a Render Object.

---

## Render Object

Example:

```json
{
  "id": "block_image_1",
  "type": "image",
  "content": {
    "url": "https://cdn.minime.ai/image.jpg",
    "alt": "Workspace photo"
  },
  "resolved_style": {}
}
```

---

## Content Structure

### url

Represents the image asset URL.

Example:

```json
{
  "url": "https://cdn.minime.ai/image.jpg"
}
```

---

### alt

Represents the image alternative text.

Example:

```json
{
  "alt": "Workspace photo"
}
```

---

## Required Fields

Required:

```text
url
```

Optional:

```text
alt
```

---

## Empty Image

If:

```text
image = null
```

or

```text
image = empty
```

the renderer must return:

```text
null
```

No Render Object should be generated.

---

## Invalid Image

Examples:

```text
Missing URL
Deleted Asset
Corrupted Asset
Invalid Asset
```

The renderer must return:

```text
null
```

---

## Alt Text Rules

If alt text exists:

```text
Return as stored
```

If alt text is missing:

```text
Return empty string
```

The renderer must not generate alt text automatically.

Example:

```json
{
  "url": "https://cdn.minime.ai/image.jpg",
  "alt": ""
}
```

---

## Content Preservation

The Image Renderer must return content exactly as stored.

The renderer must not:

```text
Compress Images
Resize Images
Crop Images
Enhance Images
Generate Alt Text
Modify Alt Text
```

These responsibilities belong elsewhere.

---

## Renderer Responsibilities

The Image Renderer may:

* Read image content
* Validate image availability
* Normalize image content
* Produce Render Objects

---

## Renderer Non-Responsibilities

The Image Renderer must not:

* Upload images
* Edit images
* Transform images
* Optimize images
* Store images
* Generate images

These responsibilities belong elsewhere.

---

## Presentation Independence

The Image Renderer must not define:

```text
Width
Height
Aspect Ratio
Border Radius
Shadow
Spacing
Alignment
Animation
```

Examples:

```text
Full Width Image
Rounded Image
Card Image
Hero Image
```

are resolved by the Block Styling System and attached to the Render Object as Resolved Style.

The renderer does not resolve or decide them.

---

## Theme Independence

The Image Renderer must not know:

```text
Theme
Layout
Template
Visual Style
```

The renderer returns content plus the Resolved Style it is given. It never reads the Theme.

---

## Multiple Image Blocks

V1 allows:

```text
Unlimited Image Blocks
```

Each Image Block is rendered independently.

Example:

```text
Image Block A
↓
Image Renderer

Image Block B
↓
Image Renderer

Image Block C
↓
Image Renderer
```

---

## Rendering Failure

The renderer returns:

```text
null
```

when image content cannot be resolved safely.

The renderer must never throw public-facing errors.

---

## Public Profile Flow

```text
Image Block
↓
Image Renderer
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
Image Resolution
Alt Text Resolution
Render Object Creation
Null-safe Rendering
```

Not Supported:

```text
Image Galleries
Image Cropping
Image Filters
Image Effects
Image Variants
AI Image Generation
Responsive Image Logic
```

---

## Final Rule

The Image Renderer answers one question:

```text
What image data should be available for presentation?
```

It does not answer:

```text
How should the Theme be resolved, or how should blocks be composed into a page?
```
