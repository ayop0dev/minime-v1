# Minime Title Block Specification V1

## Status

Approved

---

## Purpose

This document defines the Title Block for Minime V1.

It explains:

* What the Title Block is
* What content it owns
* How it behaves
* What rules apply to it
* What responsibilities it does and does not have

---

## Definition

The Title Block is a Content Block used to represent a section heading inside a profile.

The purpose of a Title Block is:

```text
Section Heading
```

Examples:

```text
About Me
Featured Links
Projects
Contact
Experience
```

The Title Block owns its own text content.

---

## Block Type

```text
title
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
title = unlimited
```

A profile may contain any number of Title Blocks.

Each Title Block is independent.

---

## Data Ownership

The Title Block owns its own content.

The source of truth is:

```text
TitleBlock.content
```

The block does not reference Profile Content, Connected Accounts, or other blocks.

---

## Required Block Fields

```ts
type TitleBlock = {
  id: string;
  account_id: string;
  type: "title";
  sort_order: number;
  content: TitleBlockContent;
  settings: TitleBlockSettings;
  created_at: string;
  updated_at: string;
};
```

---

## Content Shape

```ts
type TitleBlockContent = {
  text: string;
};
```

---

## Settings Shape

```ts
type TitleBlockSettings = {};
```

The Title Block has no block-specific settings in V1.

---

## Default Settings

```ts
const defaultTitleBlockSettings = {};
```

---

## Semantic Meaning

A Title Block represents:

```text
Heading
```

Not:

```text
Paragraph
Article
Description
Long-form content
```

The semantic purpose of the block is more important than text length.

---

## Creation Rules

Creating a Title Block requires:

```text
text
```

---

## Editing Rules

Editing a Title Block may update:

```text
text
sort_order
```

---

## Deletion Rules

Deleting a Title Block removes only that block.

No other blocks are affected.

---

## Validation Rules

A Title Block is valid when:

```text
type = title
account_id exists
sort_order is valid
content.text exists
```

---

## Empty Content Behavior

If:

```text
content.text
```

is empty or invalid:

* The profile must not break
* The block may be skipped
* The editor may show a validation warning

---

## Renderer Behavior

The Block Renderer receives:

```text
TitleBlock
+
Resolved Style
```

and reads:

```text
content.text
```

The renderer produces a Render Object and attaches the Resolved Style it is given.

The renderer must not modify block content.

---

## Presentation Rules

The Title Block does not own:

* Typography
* Font size
* Font weight
* Color
* Alignment
* Spacing
* Animation
* Theme integration

These are defined by the Theme and resolved by the Block Styling System into Resolved Style. The renderer only consumes the result.

---

## Responsibilities

The Title Block is responsible for:

* Owning title content
* Defining placement order
* Representing a section heading

---

## Non-Responsibilities

The Title Block is not responsible for:

* Long-form content
* Rich text
* HTML
* Markdown
* Theme decisions
* Rendering decisions
* Analytics
* AI functionality

---

## Relationship With Other Blocks

The Title Block has no parent-child relationship with any block.

Examples:

```text
Title
↓

Textbox
```

or

```text
Title
↓

Buttons
```

are ordering decisions only.

The blocks remain independent.

---

## V1 Exclusions

The Title Block V1 does not support:

* Rich text
* Markdown
* HTML
* Icons
* Emojis
* Images
* Nested blocks
* Child blocks
* Styling controls

---

## Canonical Example

```json
{
  "id": "block_08",
  "account_id": "account_01",
  "type": "title",
  "sort_order": 50,
  "content": {
    "text": "Featured Links"
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
Title Block = Section Heading
```

The block owns its own text content and exists independently from all other blocks.
