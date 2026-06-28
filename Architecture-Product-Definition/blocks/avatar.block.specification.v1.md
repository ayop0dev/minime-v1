# Minime Avatar Block Specification V1

## Status

Approved

---

## Purpose

This document defines the Avatar Block for Minime V1.

It explains:

* What the Avatar Block is
* What data it owns
* What data it references
* How it behaves
* What rules apply to it
* What responsibilities it does and does not have

---

## Definition

The Avatar Block is an Identity Block that displays the Account owner avatar inside the public profile block order.

The Avatar Block does not own the avatar image itself.

It references:

```text
ProfileContent.avatar
```

The Avatar Block controls whether and where the avatar appears in the profile block sequence.

---

## Block Type

```text
avatar
```

---

## Block Category

```text
Identity Block
Reference Block
Single-Instance Block
```

---

## Instance Policy

```text
avatar = max 1
```

Each profile may have only one Avatar Block.

The system must prevent creating more than one Avatar Block for the same profile.

If the Avatar Block already exists, edits must update the existing block instead of creating a new one.

---

## Data Ownership

The Avatar Block does not own avatar image data.

Avatar image data belongs to Profile Content.

```text
Profile Content
└── avatar
```

The Avatar Block only references that data.

---

## Source Of Truth

The source of truth for the avatar is:

```text
ProfileContent.avatar
```

Not:

```text
AvatarBlock.content.avatar
AvatarBlock.content.image_url
AvatarBlock.content.file
```

---

## Required Block Fields

The Avatar Block follows the global Block shape:

```ts
type AvatarBlock = {
  id: string;
  account_id: string;
  type: "avatar";
  sort_order: number;
  content: AvatarBlockContent;
  settings: AvatarBlockSettings;
  created_at: string;
  updated_at: string;
};
```

---

## Content Shape

Because the Avatar Block is a Reference Block, its content must remain minimal.

```ts
type AvatarBlockContent = {};
```

The Avatar Block must not duplicate the avatar image data.

---

## Settings Shape

```ts
type AvatarBlockSettings = {
  source: "profile_avatar";
};
```

For V1, the Avatar Block always uses:

```text
ProfileContent.avatar
```

The `source` field exists to make the reference explicit.

It must not be used to point to external images, uploaded files, or other profiles.

---

## Default Settings

```ts
const defaultAvatarBlockSettings = {
  source: "profile_avatar"
};
```

---

## Creation Rules

When a profile is created, the system may create an Avatar Block by default.

If created by default:

```text
type = avatar
content = {}
settings.source = profile_avatar
```

The exact default position is controlled by profile setup rules, not by the Avatar Block itself.

---

## Editing Rules

Editing the Avatar Block may update:

```text
sort_order
settings
```

Editing the Avatar Block must not update:

```text
ProfileContent.avatar
```

Avatar image editing belongs to Profile Content, not the Avatar Block.

---

## Deletion Rules

In V1, deleting the Avatar Block means removing the avatar from the profile block sequence.

Deleting the Avatar Block must not delete:

```text
ProfileContent.avatar
```

The avatar image remains stored in Profile Content unless removed from Profile Content directly.

---

## Empty Avatar Behavior

If the Avatar Block is active but:

```text
ProfileContent.avatar
```

is empty, missing, invalid, or unavailable:

* The public profile must not break
* The Avatar Block may be skipped
* The renderer may show a safe fallback only if approved by the Rendering Engine
* The block editor may show an internal warning

The Avatar Block must not generate or guess an avatar automatically.

---

## Validation Rules

An Avatar Block is valid when:

```text
type = avatar
account_id exists
sort_order is valid
content is empty object
settings.source = profile_avatar
```

The block is renderable only when:

```text
ProfileContent.avatar exists
ProfileContent.avatar is valid
```

A valid block may still be non-renderable if the referenced avatar data is missing.

---

## Renderer Behavior

The Block Renderer receives the Avatar Block together with the Resolved Style produced by the Block Styling System, and reads:

```text
ProfileContent.avatar
```

The renderer produces a Render Object based on:

* ProfileContent.avatar
* Avatar Block settings
* Resolved Style

The renderer must not mutate:

```text
ProfileContent.avatar
AvatarBlock.content
AvatarBlock.settings
```

---

## Presentation Rules

The Avatar Block does not own presentation decisions such as:

* Avatar shape
* Avatar size
* Border radius
* Border style
* Shadow
* Position style
* Animation
* Cropping style

These are defined by the Theme and resolved by the Block Styling System into Resolved Style. The renderer only consumes the result.

---

## Responsibilities

The Avatar Block is responsible for:

* Existing as an avatar placement block
* Referencing ProfileContent.avatar
* Defining where avatar appears in block order
* Preserving single-instance behavior

---

## Non-Responsibilities

The Avatar Block is not responsible for:

* Uploading avatar images
* Cropping avatar images
* Storing image URLs
* Storing uploaded files
* Editing ProfileContent.avatar
* Deleting ProfileContent.avatar
* Generating avatar images
* Rendering final visual appearance
* Choosing avatar shape or size
* Managing account identity
* Managing connected accounts

---

## Relationship With Profile Content

Profile Content owns:

```text
ProfileContent.avatar
```

The Avatar Block references it.

Updating ProfileContent.avatar affects the visual result of the Avatar Block wherever it appears.

Removing ProfileContent.avatar may cause the Avatar Block to become non-renderable.

---

## Relationship With Rendering Engine

The Rendering Engine:

* Consumes the Avatar Render Object (content + resolved style)
* Preserves its sort order
* Composes it into the public profile

The Rendering Engine must not read raw blocks, select renderers, or edit the referenced avatar data. Renderer selection belongs to the Renderer Registry; a non-renderable block never produces a Render Object.

---

## Relationship With Block Renderer

The Block Renderer:

* Transforms the Avatar Block into a Render Object
* Reads ProfileContent.avatar
* Attaches the Resolved Style it is given
* Handles missing avatar data safely

The Block Renderer must not own avatar content.

---

## V1 Exclusions

The Avatar Block V1 does not support:

* Multiple avatars
* Avatar galleries
* Avatar upload inside the block
* Avatar cropping inside the block
* Avatar filters
* Avatar stickers
* AI-generated avatars
* External avatar URLs inside block content
* Platform-specific avatars
* Per-block avatar styling
* Conditional avatar display

---

## Canonical Example

```json
{
  "id": "block_01",
  "account_id": "account_01",
  "type": "avatar",
  "sort_order": 1,
  "content": {},
  "settings": {
    "source": "profile_avatar"
  },
  "created_at": "2026-06-24T00:00:00Z",
  "updated_at": "2026-06-24T00:00:00Z"
}
```

---

## Final Decision

In Minime V1:

```text
Avatar Block = placement reference for ProfileContent.avatar
```

It does not own the avatar image.

It only controls whether and where the avatar appears in the profile block order.
