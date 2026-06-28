# Minime Image Block Specification V1

## Status

Approved

---

## Purpose

This document defines the Image Block for Minime V1.

It explains:

* What the Image Block is
* What content it owns
* How it behaves
* What rules apply to it
* What responsibilities it does and does not have

---

## Definition

The Image Block is a Content Block that displays an image inside the public profile.

Unlike Identity Blocks:

```text
avatar
name
bio
```

the Image Block owns its own content.

Each Image Block represents a single image.

---

## Block Type

```text
image
```

---

## Block Category

```text
Content Block
Multi-Instance Block
```

---

## Instance Policy

```text
image = unlimited
```

A profile may contain any number of Image Blocks.

Each Image Block is independent.

---

## Data Ownership

The Image Block owns its image content.

The image is stored as part of the block.

The Image Block does not reference:

```text
ProfileContent.avatar
ProfileContent.bio
ProfileContent.display_name
```

---

## Source Of Truth

The source of truth for the image is:

```text
ImageBlock.content
```

---

## Required Block Fields

The Image Block follows the global Block shape:

```ts
type ImageBlock = {
  id: string;
  account_id: string;
  type: "image";
  status: "active" | "hidden";
  sort_order: number;
  content: ImageBlockContent;
  settings: ImageBlockSettings;
  created_at: string;
  updated_at: string;
};
```

---

## Content Shape

```ts
type ImageBlockContent = {
  image_id: string;
};
```

The block stores a reference to a managed image asset.

The block must not store:

* Raw image binary
* Base64 image data
* External image URLs

---

## Settings Shape

```ts
type ImageBlockSettings = {};
```

The Image Block has no block-specific settings in V1.

---

## Default Settings

```ts
const defaultImageBlockSettings = {};
```

---

## Creation Rules

Creating an Image Block requires:

```text
image_id
```

The block may not be created without a valid image asset.

---

## Editing Rules

Editing an Image Block may update:

```text
image_id
status
sort_order
```

Editing an Image Block affects only that block.

Other Image Blocks remain unchanged.

---

## Deletion Rules

Deleting an Image Block removes the block from the profile.

Deleting the block does not automatically delete the image asset.

Asset cleanup policies are handled elsewhere.

---

## Hidden State

If status is:

```text
hidden
```

the image must not render on the public profile.

The image content remains stored.

---

## Validation Rules

An Image Block is valid when:

```text
type = image
account_id exists
status is active or hidden
sort_order is valid
content.image_id exists
```

---

## Empty Image Behavior

If:

```text
content.image_id
```

is missing, invalid, unavailable, or corrupted:

* The profile must not break
* The Image Block may be skipped
* The editor may display an internal warning

The renderer must fail gracefully.

---

## Renderer Behavior

The Block Renderer receives:

```text
ImageBlock
+
Resolved Style
```

and loads:

```text
content.image_id
```

The renderer produces a Render Object and attaches the Resolved Style it is given.

The renderer must not modify:

```text
content
settings
```

---

## Presentation Rules

The Image Block does not own:

* Width
* Height
* Border radius
* Shadow
* Frame style
* Animation
* Theme integration

These are defined by the Theme and resolved by the Block Styling System into Resolved Style. The renderer only consumes the result.

---

## Responsibilities

The Image Block is responsible for:

* Owning image content
* Referencing a valid image asset
* Defining visibility
* Defining placement order
* Remaining independent from other Image Blocks

---

## Non-Responsibilities

The Image Block is not responsible for:

* Avatar management
* Image editing
* Image cropping
* Image optimization
* Asset storage management
* Rendering decisions
* Theme decisions
* Analytics
* AI functionality

---

## Relationship With Profile Content

The Image Block owns its own content.

It does not depend on:

```text
ProfileContent.avatar
ProfileContent.display_name
ProfileContent.bio
```

---

## Relationship With Rendering Engine

The Rendering Engine:

* Consumes the Image Render Object (content + resolved style)
* Preserves order
* Composes it into the public profile

The Rendering Engine must not read raw blocks, select renderers, or mutate Render Objects. Renderer selection belongs to the Renderer Registry.

---

## Relationship With Block Renderer

The Block Renderer:

* Resolves image_id
* Produces a Render Object
* Attaches the Resolved Style it is given
* Handles missing assets safely

The Block Renderer must not own image content.

---

## V1 Exclusions

The Image Block V1 does not support:

* Image galleries
* Multiple images inside one block
* Sliders
* Carousels
* Lightboxes
* External image URLs
* Video assets
* Animated media
* Per-block styling controls
* AI-generated images

---

## Canonical Example

```json
{
  "id": "block_04",
  "account_id": "account_01",
  "type": "image",
  "status": "active",
  "sort_order": 10,
  "content": {
    "image_id": "img_001"
  },
  "settings": {},
  "created_at": "2026-06-24T00:00:00Z",
  "updated_at": "2026-06-24T00:00:00Z"
}
```

---

## Final Decision

In Minime V1:

```text
Image Block = Single Image Content Unit
```

The Image Block owns its image content.

Multiple Image Blocks may exist within the same profile.
