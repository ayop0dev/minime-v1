# Minime Bio Block Specification V1

## Status

Approved

---

## Purpose

This document defines the Bio Block for Minime V1.

It explains:

* What the Bio Block is
* What data it owns
* What data it references
* How it behaves
* What rules apply to it
* What responsibilities it does and does not have

---

## Definition

The Bio Block is an Identity Block that displays the Account owner's bio inside the public profile block sequence.

The Bio Block does not own the bio content itself.

It references:

```text
ProfileContent.bio
```

The Bio Block controls whether and where the bio appears in the profile block sequence.

---

## Block Type

```text
bio
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
bio = max 1
```

Each profile may have only one Bio Block.

The system must prevent creating more than one Bio Block for the same profile.

If the Bio Block already exists, edits must update the existing block instead of creating a new one.

---

## Data Ownership

The Bio Block does not own bio data.

Bio data belongs to Profile Content.

```text
Profile Content
└── bio
```

The Bio Block only references that data.

---

## Source Of Truth

The source of truth for the profile bio is:

```text
ProfileContent.bio
```

Not:

```text
BioBlock.content.bio
BioBlock.content.text
BioBlock.content.value
```

---

## Required Block Fields

The Bio Block follows the global Block shape:

```ts
type BioBlock = {
  id: string;
  account_id: string;
  type: "bio";
  sort_order: number;
  content: BioBlockContent;
  settings: BioBlockSettings;
  created_at: string;
  updated_at: string;
};
```

---

## Content Shape

Because the Bio Block is a Reference Block, its content must remain minimal.

```ts
type BioBlockContent = {};
```

The Bio Block must not duplicate bio content.

---

## Settings Shape

```ts
type BioBlockSettings = {
  source: "profile_bio";
};
```

For V1, the Bio Block always uses:

```text
ProfileContent.bio
```

The `source` field exists to make the reference explicit.

It must not reference external sources or other profile fields.

---

## Default Settings

```ts
const defaultBioBlockSettings = {
  source: "profile_bio"
};
```

---

## Creation Rules

When a profile is created, the system may create a Bio Block by default.

If created by default:

```text
type = bio
content = {}
settings.source = profile_bio
```

The exact default position is controlled by profile setup rules, not by the Bio Block itself.

---

## Editing Rules

Editing the Bio Block may update:

```text
sort_order
settings
```

Editing the Bio Block must not update:

```text
ProfileContent.bio
```

Bio editing belongs to Profile Content, not the Bio Block.

---

## Deletion Rules

In V1, deleting the Bio Block means removing the bio from the profile block sequence.

Deleting the Bio Block must not delete:

```text
ProfileContent.bio
```

The bio remains stored in Profile Content unless changed or removed there.

---

## Empty Bio Behavior

If the Bio Block is active but:

```text
ProfileContent.bio
```

is empty, missing, invalid, or unavailable:

* The public profile must not break
* The Bio Block may be skipped
* The editor may show an internal warning
* The renderer may apply a safe fallback only if approved by the Rendering Engine

The Bio Block must not generate bio content automatically.

---

## Validation Rules

A Bio Block is valid when:

```text
type = bio
account_id exists
sort_order is valid
content is empty object
settings.source = profile_bio
```

The block is renderable only when:

```text
ProfileContent.bio exists
ProfileContent.bio is valid
```

A valid block may still be non-renderable if the referenced bio is missing.

---

## Renderer Behavior

The Block Renderer receives the Bio Block together with the Resolved Style produced by the Block Styling System, and reads:

```text
ProfileContent.bio
```

The renderer produces a Render Object based on:

* ProfileContent.bio
* Bio Block settings
* Resolved Style

The renderer must not mutate:

```text
ProfileContent.bio
BioBlock.content
BioBlock.settings
```

---

## Presentation Rules

The Bio Block does not own presentation decisions such as:

* Font family
* Font size
* Font weight
* Text color
* Line height
* Text spacing
* Alignment
* Animation
* Visual emphasis

These are defined by the Theme and resolved by the Block Styling System into Resolved Style. The renderer only consumes the result.

---

## Responsibilities

The Bio Block is responsible for:

* Existing as a bio placement block
* Referencing ProfileContent.bio
* Defining where the bio appears in block order
* Preserving single-instance behavior

---

## Non-Responsibilities

The Bio Block is not responsible for:

* Editing the bio
* Generating bio content
* AI bio writing
* AI bio optimization
* Managing profile identity
* Managing account identity
* Rendering final visual appearance
* Choosing typography
* Choosing alignment
* Choosing colors
* Managing connected accounts

---

## Relationship With Profile Content

Profile Content owns:

```text
ProfileContent.bio
```

The Bio Block references it.

Updating ProfileContent.bio affects the visual result of the Bio Block wherever it appears.

Removing ProfileContent.bio may cause the Bio Block to become non-renderable.

---

## Relationship With Rendering Engine

The Rendering Engine:

* Consumes the Bio Render Object (content + resolved style)
* Preserves its sort order
* Composes it into the public profile

The Rendering Engine must not read raw blocks, select renderers, or edit the referenced bio. Renderer selection belongs to the Renderer Registry; a non-renderable block never produces a Render Object.

---

## Relationship With Block Renderer

The Block Renderer:

* Transforms the Bio Block into a Render Object
* Reads ProfileContent.bio
* Attaches the Resolved Style it is given
* Handles missing bio safely

The Block Renderer must not own bio content.

---

## V1 Exclusions

The Bio Block V1 does not support:

* Multiple bios
* Alternate bios
* Expandable bios
* Rich text bios
* Markdown bios
* HTML bios
* Dynamic bios
* AI-generated bios
* Conditional bio display
* Per-block typography settings
* Per-block styling controls

---

## Canonical Example

```json
{
  "id": "block_03",
  "account_id": "account_01",
  "type": "bio",
  "sort_order": 3,
  "content": {},
  "settings": {
    "source": "profile_bio"
  },
  "created_at": "2026-06-24T00:00:00Z",
  "updated_at": "2026-06-24T00:00:00Z"
}
```

---

## Final Decision

In Minime V1:

```text
Bio Block = placement reference for ProfileContent.bio
```

It does not own the bio content.

It only controls whether and where the bio appears in the profile block order.
