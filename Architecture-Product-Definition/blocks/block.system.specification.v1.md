# Minime Block System Specification V1

## Status

Approved

---

## Purpose

This document defines the Minime Block System for V1.

It answers:

* What is a Block?
* What fields does every Block have?
* What rules apply to all Blocks?
* What responsibilities belong to Blocks?
* What responsibilities do not belong to Blocks?
* Which block types are supported in V1?
* Which block types are single-instance or unlimited?

This document does not define the internal behavior of each individual block type.

Individual block specifications are defined separately.

---

## Definition

A Block is a single content unit that belongs directly to an Account.

A Block represents something the Account owner wants to show on their public profile.

Examples:

* Avatar
* Name
* Bio
* Image
* Button
* Social Icons
* Divider
* Title
* Textbox

In Minime V1:

```text
Profile
↓
Blocks
```

There are no pages, subpages, or multi-page profile structures.

---

## Core Principle

A Block is content.

A Block is not layout.

A Block is not a renderer.

A Block is not a theme.

A Block is not responsible for deciding how the public profile looks.

---

## Ownership

Blocks are owned by Profile Content.

Profile Content owns:

```text
display_name
avatar
bio
contact_information
blocks
```

Each Block belongs directly to one Account.

Blocks do not belong to Pages.

Blocks do not belong to the Rendering Domain.

Blocks do not belong to the Rendering Engine.

---

## Required Fields For Every Block

Every Block must have the following fields:

```ts
id
account_id
type
status
sort_order
content
settings
created_at
updated_at
```

---

## Field Definitions

### id

Unique identifier for the block.

Used internally to reference, update, reorder, delete, and render the block.

---

### account_id

The Account that owns this block.

Each block belongs to exactly one Account.

---

### type

Defines the block type.

Allowed V1 block types:

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

The type determines which block specification applies.

---

### status

Defines whether the block should be used.

Allowed V1 statuses:

```text
active
hidden
```

#### active

The block is visible and eligible for rendering.

#### hidden

The block remains saved but is not rendered on the public profile.

---

### sort_order

Defines the block position inside the profile.

The profile renders blocks according to `sort_order`.

Blocks may be reordered by the user.

The block itself does not decide its position.

---

### content

The actual user-owned content of the block.

Examples:

* Avatar image reference
* Name text
* Bio text
* Button label
* Button URL
* Image reference
* Social account list
* Divider intent
* Title text
* Textbox content

The structure of `content` depends on the block type.

---

### settings

Block-specific user preferences that are still considered content-level configuration.

Examples:

* Whether a button opens in a new tab
* Whether social icons show labels
* Whether a divider has a label
* Text alignment when explicitly controlled by the user

Settings must not contain theme-level styling decisions.

---

### created_at

Timestamp of block creation.

---

### updated_at

Timestamp of last block update.

---

## V1 Block Types

The approved V1 block types are:

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

No other block types are supported in V1.

---

## Block Categories

For V1, blocks are divided into two logical categories:

```text
Identity Blocks
Content Blocks
```

This categorization helps define instance rules only.

It does not change ownership.

All blocks still belong directly to the Account.

---

## Identity Blocks

Identity Blocks represent the main public identity of the Account owner.

V1 Identity Blocks are:

```text
avatar
name
bio
```

Identity Blocks are single-instance blocks.

Each profile may have only one active or saved instance of each Identity Block type.

---

## Content Blocks

Content Blocks represent flexible profile content.

V1 Content Blocks are:

```text
image
social_icons
button
divider
title
textbox
```

Content Blocks may be repeated according to their instance policy.

---

## Block Instance Policy

Each block type has a defined instance policy.

```text
avatar        = max 1
name          = max 1
bio           = max 1

image         = unlimited
social_icons  = unlimited
button        = unlimited
divider       = unlimited
title         = unlimited
textbox       = unlimited
```

---

## Single-Instance Block Rule

A single-instance block means:

```text
Only one block of this type may exist per profile.
```

This applies to:

```text
avatar
name
bio
```

The system must prevent creating duplicate blocks of these types for the same profile.

If a user edits one of these blocks, the existing block is updated.

A new block is not created.

---

## Unlimited Block Rule

An unlimited block means:

```text
The profile may contain multiple blocks of this type.
```

This applies to:

```text
image
social_icons
button
divider
title
textbox
```

Each instance has its own:

```text
id
sort_order
content
settings
status
created_at
updated_at
```

---

## Global Block Rules

### 1. Blocks Belong Directly To Account

In V1, blocks are stored directly under the Account.

```text
Account.blocks[]
```

Not:

```text
Account.pages[].blocks[]
```

Pages are not supported in V1.

---

### 2. Blocks Are Ordered Content Units

The order of blocks is part of Profile Content.

The user may reorder blocks.

The profile-defined order is carried through to the Render Objects, and the Rendering Engine composes the public profile in that order.

---

### 3. Blocks Do Not Render Themselves

A block does not decide its final visual output.

Rendering is handled by the Block Renderer, which transforms the block into a Render Object.

```text
Block = Content
Block Renderer = Render Object
```

---

### 4. Blocks Do Not Own Appearance

Blocks must not own:

* Theme
* Colors
* Typography system
* Spacing system
* Animation behavior
* Global profile layout
* Public page composition

Appearance is defined by the Theme and resolved by the Block Styling System; layout and page composition belong to the Rendering Engine.

---

### 5. Blocks May Contain Limited User Preferences

A block may contain settings only when they are directly related to the meaning or behavior of that block.

Example:

A button may store:

```text
label
url
open_in_new_tab
```

But it must not store:

```text
button_color
font_family
global_spacing
theme_style
```

Unless explicitly approved in a block-specific specification.

---

### 6. Blocks Must Be Independently Validatable

Each block type must define its own validation rules.

The system must be able to validate a block without needing to render the full profile.

---

### 7. Invalid Blocks Must Not Break The Profile

If a block is invalid, missing required content, or cannot be rendered safely:

* The profile must still load
* Other valid blocks must still render
* The invalid block may be skipped
* The system may show an internal editor warning

Public rendering must fail gracefully.

---

### 8. Blocks Must Be Stable Over Time

A block should preserve its identity even when its content changes.

Editing a block should not create a new block unless the user explicitly adds a new one.

---

### 9. Blocks Must Be Portable Within The Same Profile

A block can be reordered within the same profile.

In V1, moving blocks between profiles is not supported unless explicitly added later.

---

### 10. Blocks Must Be Renderer-Agnostic

A block must not depend on a specific renderer implementation.

The same block content should be usable by different renderers in the future.

---

### 11. Blocks Must Respect Their Instance Policy

The system must enforce each block type instance policy.

Single-instance blocks must not be duplicated.

Unlimited blocks may be created multiple times.

---

### 12. Block Count Ceiling

"Unlimited" in the Block Instance Policy means there is no product-level reason to restrict count for that block type.

The Block System defines that a maximum total block count per profile must exist. This ceiling exists for system stability, not for product reasons. The Block System owns this constraint.

The specific ceiling value is a configuration value that may be supplied at runtime. Implementation must enforce this ceiling at the API validation layer.

Any profile that reaches the ceiling must be prevented from adding further blocks until existing blocks are deleted.

---

## Block Responsibilities

Blocks are responsible for storing:

* Block identity
* Block type
* Account ownership
* Block status
* Block order
* Block content
* Block-level settings
* Creation and update timestamps

Blocks are also responsible for being validatable according to their block type.

---

## Responsibilities That Do Not Belong To Blocks

Blocks are not responsible for:

* Rendering visual output
* Choosing themes
* Choosing global colors
* Choosing typography
* Controlling full profile layout
* Managing profile composition
* Editing account data
* Managing authentication
* Managing connected accounts
* Tracking analytics
* Handling payments
* Handling AI decisions
* Handling QR codes
* Handling UTM rules
* Managing public routing
* Creating pages or subpages

---

## Relationship With Rendering Engine

The Rendering Engine receives Render Objects (content + resolved style) and composes the public profile.

The Rendering Engine may:

* Consume Render Objects
* Preserve profile-defined block order
* Compose the final public profile output

The Rendering Engine must not mutate Render Objects. It does not read raw blocks and it does not select renderers — renderer selection belongs to the Renderer Registry.

---

## Relationship With Block Renderer

The Block Renderer receives Block Data and the Resolved Style produced by the Block Styling System, and produces a Render Object.

The Block Renderer may interpret:

* Block type
* Block content
* Block settings
* Resolved Style

The Block Renderer must not:

* Edit block content
* Reorder blocks
* Save user data
* Own block data
* Override user content

---

## Relationship With Rendering

The Rendering Domain composes the page structure.

Appearance is defined by the Theme and resolved by the Block Styling System; the result is carried in each Render Object's Resolved Style.

Examples of appearance owned upstream (Appearance Domain + Block Styling), not by the block:

* Theme
* Background
* Typography
* Colors
* Border radius
* Spacing
* Visual style

The Rendering Domain does not resolve or apply styles, and it does not own block content.

---

## Relationship With Account Management

Account Management owns account-level and content-level data.

It may create, update, hide, delete, or reorder blocks through Profile Content flows.

Account Management does not render blocks.

---

## V1 Exclusions

The Block System V1 does not support:

* Pages
* Subpages
* Multi-page profiles
* Nested blocks
* Block groups
* Conditional blocks
* Scheduled blocks
* Dynamic AI-generated blocks
* Analytics-owned blocks
* Payment blocks
* Product blocks
* Form blocks
* Embed blocks

These may be considered in future versions, but they are out of scope for V1.

---

## Canonical Block Shape

```ts
type Block = {
  id: string;
  account_id: string;
  type: BlockType;
  status: BlockStatus;
  sort_order: number;
  content: Record<string, unknown>;
  settings: Record<string, unknown>;
  created_at: string;
  updated_at: string;
};
```

---

## Canonical Block Type

```ts
type BlockType =
  | "avatar"
  | "name"
  | "bio"
  | "image"
  | "social_icons"
  | "button"
  | "divider"
  | "title"
  | "textbox";
```

---

## Canonical Block Status

```ts
type BlockStatus =
  | "active"
  | "hidden";
```

---

## Canonical Block Instance Policy

```ts
type BlockInstancePolicy = {
  avatar: "max_1";
  name: "max_1";
  bio: "max_1";

  image: "unlimited";
  social_icons: "unlimited";
  button: "unlimited";
  divider: "unlimited";
  title: "unlimited";
  textbox: "unlimited";
};
```

---

## System Invariant

For Minime V1, the profile structure must always remain:

```text
Profile
↓
Blocks
↓
Block Renderer
↓
Render Object
↓
Rendering Engine
↓
Public Profile
```

Blocks must never introduce pages, subpages, or layout ownership.

---

## Final Decision

In Minime V1:

```text
Block = Account-owned content unit
```

```text
Renderer = Render Object
```

```text
Presentation = Page composition
```

```text
Rendering Engine = Composition
```

A Block stores what should exist.

It does not decide how the profile should look.
