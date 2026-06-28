# Minime Button Block Specification V1

## Status

Approved

---

## Purpose

This document defines the Button Block for Minime V1.

It explains:

* What the Button Block is
* What content it owns
* How it behaves
* What rules apply to it
* What responsibilities it does and does not have

---

## Definition

The Button Block is a Content Block that displays a clickable destination inside the public profile.

Each Button Block represents a single button.

A Button Block owns its own label and destination URL.

---

## Block Type

```text
button
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
button = unlimited
```

A profile may contain any number of Button Blocks.

Each Button Block is independent.

---

## Data Ownership

The Button Block owns its own content.

The source of truth is:

```text
ButtonBlock.content
```

The Button Block does not reference:

```text
Profile Content
Connected Accounts
Other Blocks
```

---

## Required Block Fields

The Button Block follows the global Block shape:

```ts
type ButtonBlock = {
  id: string;
  account_id: string;
  type: "button";
  sort_order: number;
  content: ButtonBlockContent;
  settings: ButtonBlockSettings;
  created_at: string;
  updated_at: string;
};
```

---

## Content Shape

```ts
type ButtonBlockContent = {
  label: string;
  url: string;
};
```

---

## Content Ownership

The Button Block owns:

```text
label
url
```

The Button Block must not depend on external entities to resolve its destination.

---

## Settings Shape

```ts
type ButtonBlockSettings = {};
```

The Button Block has no block-specific settings in V1.

---

## Default Settings

```ts
const defaultButtonBlockSettings = {};
```

---

## Creation Rules

Creating a Button Block requires:

```text
label
url
```

The block may not be created without both fields.

---

## Label Rules

The label is the visible text displayed on the button.

Examples:

```text
Visit Website
Book A Call
Download CV
View Portfolio
```

The label must be plain text.

---

## URL Rules

The URL defines the button destination.

Examples:

```text
https://example.com
https://linkedin.com/in/example
mailto:user@example.com
tel:+201234567890
```

The URL must be stored exactly as entered after validation.

---

## Description Policy

The Button Block does not support:

```text
subtitle
description
secondary text
helper text
caption
```

If explanatory text is needed, it must be represented using:

```text
Title Block
Textbox Block
```

outside the Button Block.

---

## Editing Rules

Editing a Button Block may update:

```text
label
url
sort_order
```

Editing one Button Block must not affect any other Button Block.

---

## Deletion Rules

Deleting a Button Block removes the block from the profile.

Deletion removes only that block.

No other blocks are affected.

---

## Validation Rules

A Button Block is valid when:

```text
type = button
account_id exists
sort_order is valid
label exists
url exists
```

---

## Empty Content Behavior

If:

```text
label
```

or

```text
url
```

is missing, invalid, or empty:

* The profile must not break
* The block may be skipped
* The editor may show a validation warning

The renderer must fail gracefully.

---

## Renderer Behavior

The Block Renderer receives:

```text
ButtonBlock
+
Resolved Style
```

and reads:

```text
content.label
content.url
```

The renderer produces a Render Object and attaches the Resolved Style it is given.

The renderer must not modify:

```text
content
settings
```

---

## Presentation Rules

The Button Block does not own:

* Colors
* Typography
* Border radius
* Shadows
* Hover effects
* Animation
* Width
* Height
* Theme integration

These are defined by the Theme and resolved by the Block Styling System into Resolved Style. The renderer only consumes the result.

---

## Responsibilities

The Button Block is responsible for:

* Owning button content
* Owning destination URL
* Defining placement order
* Remaining independent from other Button Blocks

---

## Non-Responsibilities

The Button Block is not responsible for:

* URL shortening
* Analytics
* Click tracking
* UTM management
* QR generation
* Theme decisions
* Rendering decisions
* AI functionality

---

## Relationship With Profile Content

The Button Block is self-contained.

It does not depend on:

```text
ProfileContent.avatar
ProfileContent.display_name
ProfileContent.bio
```

---

## Relationship With Connected Accounts

The Button Block does not depend on Connected Accounts.

A button may point to any valid destination URL.

---

## Relationship With Rendering Engine

The Rendering Engine:

* Consumes the Button Render Object (content + resolved style)
* Preserves order
* Composes it into the public profile

The Rendering Engine must not read raw blocks, select renderers, or mutate Render Objects. Renderer selection belongs to the Renderer Registry.

---

## Relationship With Block Renderer

The Block Renderer:

* Produces a Render Object
* Uses label and URL
* Attaches the Resolved Style it is given
* Handles invalid destinations safely

The Block Renderer must not own button content.

---

## V1 Exclusions

The Button Block V1 does not support:

* Button descriptions
* Multi-line buttons
* Multiple URLs
* Conditional destinations
* Dynamic destinations
* Product links
* Booking integrations
* Form integrations
* Per-button styling controls

---

## Canonical Example

```json
{
  "id": "block_06",
  "account_id": "account_01",
  "type": "button",
  "sort_order": 30,
  "content": {
    "label": "Visit Website",
    "url": "https://example.com"
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
Button Block
=
Label
+
URL
```

The Button Block owns its own destination.

Each Button Block represents exactly one clickable action.
