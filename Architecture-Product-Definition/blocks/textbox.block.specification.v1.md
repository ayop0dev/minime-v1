# Minime Textbox Block Specification V1

## Status

Approved

---

## Purpose

This document defines the Textbox Block for Minime V1.

It explains:

* What the Textbox Block is
* What content it owns
* How it behaves
* What rules apply to it
* What responsibilities it does and does not have

---

## Definition

The Textbox Block is a Content Block used to represent section content inside a profile.

The purpose of a Textbox Block is:

```text
Section Content
```

Examples:

```text
Personal introduction
Project description
Professional summary
Custom content
```

The Textbox Block owns its own text content.

---

## Block Type

```text
textbox
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
textbox = unlimited
```

A profile may contain any number of Textbox Blocks.

Each Textbox Block is independent.

---

## Data Ownership

The Textbox Block owns its own content.

The source of truth is:

```text
TextboxBlock.content
```

The block does not reference Profile Content, Connected Accounts, or other blocks.

---

## Required Block Fields

```ts
type TextboxBlock = {
  id: string;
  account_id: string;
  type: "textbox";
  status: "active" | "hidden";
  sort_order: number;
  content: TextboxBlockContent;
  settings: TextboxBlockSettings;
  created_at: string;
  updated_at: string;
};
```

---

## Content Shape

```ts
type TextboxBlockContent = {
  text: string;
};
```

---

## Settings Shape

```ts
type TextboxBlockSettings = {};
```

The Textbox Block has no block-specific settings in V1.

---

## Default Settings

```ts
const defaultTextboxBlockSettings = {};
```

---

## Semantic Meaning

A Textbox Block represents:

```text
Content
```

Not:

```text
Heading
Section Title
Label
```

The semantic purpose of the block is more important than text length.

---

## Creation Rules

Creating a Textbox Block requires:

```text
text
```

---

## Editing Rules

Editing a Textbox Block may update:

```text
text
status
sort_order
```

---

## Deletion Rules

Deleting a Textbox Block removes only that block.

No other blocks are affected.

---

## Hidden State

If status is:

```text
hidden
```

the Textbox Block must not render.

The content remains stored.

---

## Validation Rules

A Textbox Block is valid when:

```text
type = textbox
account_id exists
status is active or hidden
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
TextboxBlock
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

The Textbox Block does not own:

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

The Textbox Block is responsible for:

* Owning text content
* Defining visibility
* Defining placement order
* Representing content

---

## Non-Responsibilities

The Textbox Block is not responsible for:

* Headings
* Rich text
* HTML
* Markdown
* Theme decisions
* Rendering decisions
* Analytics
* AI functionality

---

## Relationship With Other Blocks

The Textbox Block has no parent-child relationship with any block.

Examples:

```text
Title
↓

Textbox
```

or

```text
Textbox
↓

Button
```

are ordering decisions only.

The blocks remain independent.

---

## V1 Exclusions

The Textbox Block V1 does not support:

* Rich text
* Markdown
* HTML
* Images
* Embedded media
* Nested blocks
* Child blocks
* Styling controls

---

## Canonical Example

```json
{
  "id": "block_09",
  "account_id": "account_01",
  "type": "textbox",
  "status": "active",
  "sort_order": 60,
  "content": {
    "text": "I build brands, products, and digital experiences."
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
Textbox Block = Section Content
```

The block owns its own text content and exists independently from all other blocks.
