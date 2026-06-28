# Minime Name Block Specification V1

## Status

Approved

---

## Purpose

This document defines the Name Block for Minime V1.

It explains:

* What the Name Block is
* What data it owns
* What data it references
* How it behaves
* What rules apply to it
* What responsibilities it does and does not have

---

## Definition

The Name Block is an Identity Block that displays the Account owner's display name inside the public profile block sequence.

The Name Block does not own the name itself.

It references:

```text
ProfileContent.display_name
```

The Name Block controls whether and where the display name appears in the profile block sequence.

---

## Block Type

```text
name
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
name = max 1
```

Each profile may have only one Name Block.

The system must prevent creating more than one Name Block for the same profile.

If the Name Block already exists, edits must update the existing block instead of creating a new one.

---

## Data Ownership

The Name Block does not own display name data.

Display name data belongs to Profile Content.

```text
Profile Content
└── display_name
```

The Name Block only references that data.

---

## Source Of Truth

The source of truth for the profile name is:

```text
ProfileContent.display_name
```

Not:

```text
NameBlock.content.name
NameBlock.content.value
NameBlock.content.text
```

---

## Required Block Fields

The Name Block follows the global Block shape:

```ts
type NameBlock = {
  id: string;
  account_id: string;
  type: "name";
  sort_order: number;
  content: NameBlockContent;
  settings: NameBlockSettings;
  created_at: string;
  updated_at: string;
};
```

---

## Content Shape

Because the Name Block is a Reference Block, its content must remain minimal.

```ts
type NameBlockContent = {};
```

The Name Block must not duplicate the display name value.

---

## Settings Shape

```ts
type NameBlockSettings = {
  source: "profile_display_name";
};
```

For V1, the Name Block always uses:

```text
ProfileContent.display_name
```

The `source` field exists to make the reference explicit.

It must not reference external sources or other profile fields.

---

## Default Settings

```ts
const defaultNameBlockSettings = {
  source: "profile_display_name"
};
```

---

## Creation Rules

When a profile is created, the system may create a Name Block by default.

If created by default:

```text
type = name
content = {}
settings.source = profile_display_name
```

The exact default position is controlled by profile setup rules, not by the Name Block itself.

---

## Editing Rules

Editing the Name Block may update:

```text
sort_order
settings
```

Editing the Name Block must not update:

```text
ProfileContent.display_name
```

Display name editing belongs to Profile Content, not the Name Block.

---

## Deletion Rules

In V1, deleting the Name Block means removing the display name from the profile block sequence.

Deleting the Name Block must not delete:

```text
ProfileContent.display_name
```

The display name remains stored in Profile Content unless changed or removed there.

---

## Empty Name Behavior

If the Name Block is active but:

```text
ProfileContent.display_name
```

is empty, missing, invalid, or unavailable:

* The public profile must not break
* The Name Block may be skipped
* The editor may show an internal warning
* The renderer may apply a safe fallback only if approved by the Rendering Engine

The Name Block must not generate a display name automatically.

---

## Validation Rules

A Name Block is valid when:

```text
type = name
account_id exists
sort_order is valid
content is empty object
settings.source = profile_display_name
```

The block is renderable only when:

```text
ProfileContent.display_name exists
ProfileContent.display_name is valid
```

A valid block may still be non-renderable if the referenced display name is missing.

---

## Renderer Behavior

The Block Renderer receives the Name Block together with the Resolved Style produced by the Block Styling System, and reads:

```text
ProfileContent.display_name
```

The renderer produces a Render Object based on:

* ProfileContent.display_name
* Name Block settings
* Resolved Style

The renderer must not mutate:

```text
ProfileContent.display_name
NameBlock.content
NameBlock.settings
```

---

## Presentation Rules

The Name Block does not own presentation decisions such as:

* Font family
* Font size
* Font weight
* Text color
* Letter spacing
* Alignment
* Animation
* Visual emphasis

These are defined by the Theme and resolved by the Block Styling System into Resolved Style. The renderer only consumes the result.

---

## Responsibilities

The Name Block is responsible for:

* Existing as a display name placement block
* Referencing ProfileContent.display_name
* Defining where the name appears in block order
* Preserving single-instance behavior

---

## Non-Responsibilities

The Name Block is not responsible for:

* Editing the display name
* Generating a display name
* Validating username availability
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
ProfileContent.display_name
```

The Name Block references it.

Updating ProfileContent.display_name affects the visual result of the Name Block wherever it appears.

Removing ProfileContent.display_name may cause the Name Block to become non-renderable.

---

## Relationship With Rendering Engine

The Rendering Engine:

* Consumes the Name Render Object (content + resolved style)
* Preserves its sort order
* Composes it into the public profile

The Rendering Engine must not read raw blocks, select renderers, or edit the referenced display name. Renderer selection belongs to the Renderer Registry; a non-renderable block never produces a Render Object.

---

## Relationship With Block Renderer

The Block Renderer:

* Transforms the Name Block into a Render Object
* Reads ProfileContent.display_name
* Attaches the Resolved Style it is given
* Handles missing display name safely

The Block Renderer must not own display name content.

---

## V1 Exclusions

The Name Block V1 does not support:

* Multiple names
* Alternate names
* Nickname switching
* Localized names
* Dynamic names
* AI-generated names
* Conditional display names
* Per-block typography settings
* Per-block styling controls

---

## Canonical Example

```json
{
  "id": "block_02",
  "account_id": "account_01",
  "type": "name",
  "sort_order": 2,
  "content": {},
  "settings": {
    "source": "profile_display_name"
  },
  "created_at": "2026-06-24T00:00:00Z",
  "updated_at": "2026-06-24T00:00:00Z"
}
```

---

## Final Decision

In Minime V1:

```text
Name Block = placement reference for ProfileContent.display_name
```

It does not own the display name.

It only controls whether and where the display name appears in the profile block order.
