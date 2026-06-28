# Minime Divider Block Specification V1

## Status

Approved

---

## Purpose

This document defines the Divider Block for Minime V1.

It explains:

* What the Divider Block is
* What content it owns
* How it behaves
* What rules apply to it
* What responsibilities it does and does not have

---

## Definition

The Divider Block is a Content Block used to visually separate content sections inside a profile.

The Divider Block does not display content.

Its purpose is only:

```text
Visual Separation
```

The Divider Block may appear anywhere within the profile block sequence.

---

## Block Type

```text
divider
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
divider = unlimited
```

A profile may contain any number of Divider Blocks.

Each Divider Block is independent.

---

## Data Ownership

The Divider Block owns its own existence.

The Divider Block does not reference:

```text
Profile Content
Connected Accounts
Other Blocks
```

---

## Source Of Truth

The source of truth is:

```text
DividerBlock
```

The block is self-contained.

---

## Required Block Fields

The Divider Block follows the global Block shape:

```ts
type DividerBlock = {
  id: string;
  account_id: string;
  type: "divider";
  sort_order: number;
  content: DividerBlockContent;
  settings: DividerBlockSettings;
  created_at: string;
  updated_at: string;
};
```

---

## Content Shape

```ts
type DividerBlockContent = {};
```

The Divider Block contains no user content in V1.

---

## Settings Shape

```ts
type DividerBlockSettings = {};
```

The Divider Block contains no block-specific settings in V1.

---

## Default Settings

```ts
const defaultDividerBlockSettings = {};
```

---

## Creation Rules

A Divider Block may be created without additional data.

Creation requires only:

```text
type = divider
```

---

## Editing Rules

Editing a Divider Block may update:

```text
sort_order
```

No content fields exist to edit.

---

## Deletion Rules

Deleting a Divider Block removes only that Divider Block.

No other blocks are affected.

---

## Validation Rules

A Divider Block is valid when:

```text
type = divider
account_id exists
sort_order is valid
content is empty object
```

---

## Empty Content Behavior

The Divider Block does not require content.

The following is valid:

```ts
content = {}
```

---

## Renderer Behavior

The Block Renderer receives:

```text
DividerBlock
+
Resolved Style
```

and produces a Render Object representing a visual separator, attaching the Resolved Style it is given.

The renderer must not modify:

```text
content
settings
```

---

## Presentation Rules

The Divider Block does not own:

* Line style
* Thickness
* Width
* Color
* Margin
* Padding
* Animation
* Theme integration

These are defined by the Theme and resolved by the Block Styling System into Resolved Style. The renderer only consumes the result.

---

## Divider Policy

In V1, a Divider Block is:

```text
Visual Only
```

The Divider Block does not support:

```text
label
title
caption
text
icon
emoji
badge
```

If a section title is required, it must be represented using:

```text
Title Block
```

not Divider Block.

---

## Responsibilities

The Divider Block is responsible for:

* Creating visual separation
* Defining placement order
* Existing as an independent block

---

## Non-Responsibilities

The Divider Block is not responsible for:

* Displaying text
* Displaying labels
* Displaying icons
* Displaying content
* Managing layout
* Managing themes
* Analytics
* AI functionality
* Rendering decisions

---

## Relationship With Profile Content

The Divider Block is self-contained.

It does not depend on:

```text
ProfileContent.avatar
ProfileContent.display_name
ProfileContent.bio
```

---

## Relationship With Connected Accounts

The Divider Block has no relationship with Connected Accounts.

---

## Relationship With Rendering Engine

The Rendering Engine:

* Consumes the Divider Render Object (content + resolved style)
* Preserves order
* Composes it into the public profile

The Rendering Engine must not read raw blocks, select renderers, or mutate Render Objects. Renderer selection belongs to the Renderer Registry.

---

## Relationship With Block Renderer

The Block Renderer:

* Produces a Render Object
* Attaches the Resolved Style it is given
* Handles rendering safely

The Block Renderer must not own block data.

---

## V1 Exclusions

The Divider Block V1 does not support:

* Labels
* Titles
* Captions
* Icons
* Emojis
* Badges
* Multiple divider styles
* Per-divider styling controls
* Interactive behavior

---

## Canonical Example

```json
{
  "id": "block_07",
  "account_id": "account_01",
  "type": "divider",
  "sort_order": 40,
  "content": {},
  "settings": {},
  "created_at": "2026-06-24T00:00:00Z",
  "updated_at": "2026-06-24T00:00:00Z"
}
```

---

## Final Decision

In Minime V1:

```text
Divider Block
=
Visual Separation Only
```

The Divider Block owns no content.

It exists only to create visual separation between profile sections.
