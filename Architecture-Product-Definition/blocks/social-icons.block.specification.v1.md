# Minime Social Icons Block Specification V1

## Status

Approved

---

## Purpose

This document defines the Social Icons Block for Minime V1.

It explains:

* What the Social Icons Block is
* What data it owns
* What data it references
* How it behaves
* What rules apply to it
* What responsibilities it does and does not have

---

## Definition

The Social Icons Block is a Collection Reference Block that displays one or more connected social accounts inside the public profile.

The Social Icons Block does not own social account data.

It references accounts that already exist in:

```text
Connected Accounts
```

The block controls:

* Which connected accounts appear
* The order in which they appear

The block does not own platform usernames, URLs, or account metadata.

---

## Block Type

```text
social_icons
```

---

## Block Category

```text
Collection Reference Block
Multi-Instance Block
```

---

## Instance Policy

```text
social_icons = unlimited
```

A profile may contain any number of Social Icons Blocks.

Each block is independent.

Each block may display a different set of connected accounts.

---

## Data Ownership

The Social Icons Block does not own social account data.

Social account data belongs to:

```text
Connected Accounts
```

The block only references those accounts.

---

## Source Of Truth

The source of truth for social account information is the user-provided social account record, persisted in:

```text
Connected Accounts
```

Connected Accounts is the canonical store of that user-provided record. The user remains the source of truth; the block never overrides user intent.

Not:

```text
SocialIconsBlock.content.url
SocialIconsBlock.content.username
SocialIconsBlock.content.platform
```

---

## Required Block Fields

The Social Icons Block follows the global Block shape:

```ts
type SocialIconsBlock = {
  id: string;
  account_id: string;
  type: "social_icons";
  sort_order: number;
  content: SocialIconsBlockContent;
  settings: SocialIconsBlockSettings;
  created_at: string;
  updated_at: string;
};
```

---

## Content Shape

```ts
type SocialIconsBlockContent = {
  accounts: SocialIconsBlockAccount[];
};
```

---

## Account Shape

```ts
type SocialIconsBlockAccount = {
  connected_account_id: string;
  sort_order: number;
};
```

---

## Content Ownership

The block owns:

```text
account selection
account ordering
```

The block does not own:

```text
platform
username
url
display name
profile metadata
```

Those remain owned by Connected Accounts.

---

## Settings Shape

```ts
type SocialIconsBlockSettings = {};
```

The Social Icons Block has no block-specific settings in V1.

---

## Default Settings

```ts
const defaultSocialIconsBlockSettings = {};
```

---

## Creation Rules

A Social Icons Block may be created even if no connected accounts exist.

In that case:

```ts
{
  accounts: []
}
```

is valid.

The block simply renders nothing until accounts are added.

---

## Account Selection Rules

The block may reference:

```text
0
1
many
```

connected accounts.

The user decides which accounts appear inside each Social Icons Block.

The block never automatically displays all connected accounts.

---

## Account Ordering Rules

Accounts inside the block have their own ordering.

Example:

```text
Instagram
LinkedIn
GitHub
```

or

```text
GitHub
LinkedIn
Instagram
```

The block controls this order.

Connected Accounts does not.

---

## Editing Rules

Editing a Social Icons Block may update:

```text
accounts
account order
sort_order
```

Editing the block must not modify:

```text
Connected Account URL
Connected Account Username
Connected Account Platform
Connected Account Metadata
```

Those belong to Connected Accounts.

---

## Deletion Rules

Deleting a Social Icons Block removes the block from the profile.

Deleting the block must not delete:

```text
Connected Accounts
```

All connected accounts remain intact.

---

## Empty Block Behavior

If:

```text
accounts = []
```

the profile must still function correctly.

The block may be skipped during rendering.

The editor may show an internal warning or informational state.

---

## Validation Rules

A Social Icons Block is valid when:

```text
type = social_icons
account_id exists
sort_order is valid
accounts is an array
```

Each account entry is valid when:

```text
connected_account_id exists
sort_order is valid
```

---

## Missing Account Behavior

If a referenced:

```text
connected_account_id
```

no longer exists:

* The profile must not break
* That account item may be skipped
* Other account items must continue rendering
* The editor may show a warning

---

## Renderer Behavior

The Block Renderer receives:

```text
SocialIconsBlock
+
Resolved Style
```

It resolves:

```text
connected_account_id
```

through Connected Accounts.

The renderer may read:

```text
platform
url
username
```

from Connected Accounts.

The renderer produces a Render Object and attaches the Resolved Style it is given.

The renderer must not modify:

```text
Connected Accounts
SocialIconsBlock.content
SocialIconsBlock.settings
```

---

## Presentation Rules

The Social Icons Block does not own:

* Icon style
* Icon size
* Icon color
* Hover effects
* Animation
* Layout style
* Theme integration

These are defined by the Theme and resolved by the Block Styling System into Resolved Style. The renderer only consumes the result.

---

## Responsibilities

The Social Icons Block is responsible for:

* Referencing connected accounts
* Selecting which accounts appear
* Defining account order
* Defining block placement order

---

## Non-Responsibilities

The Social Icons Block is not responsible for:

* Creating connected accounts
* Collecting or normalizing social accounts
* Verifying social accounts
* Storing usernames
* Storing URLs
* Updating account metadata
* Rendering visual appearance
* Managing themes
* Managing analytics
* Managing AI functionality

---

## Relationship With Connected Accounts

Connected Accounts owns:

```text
platform
username
url
metadata
```

The Social Icons Block references them.

Changes to Connected Accounts automatically affect any Social Icons Block that references them.

---

## Cross-Domain Contract: Connected Accounts

| Role | Domain |
|---|---|
| Canonical Storage | Connected Accounts |
| Consumer | Social Icons Block |

The Social Icons Block is a read-only consumer of Connected Accounts.

**Ownership:**

Connected Accounts owns platform, username, and url.

The Social Icons Block owns account selection and ordering within the block.

**Write Authority:**

The Social Icons Block must not write to Connected Accounts.

Editing a Social Icons Block must not modify any Connected Account record.

**Read Authority:**

The Social Icons Block renderer reads platform, username, and url from Connected Accounts by resolving `connected_account_id` at render time.

**Ownership Traceability:**

User input → Social Accounts Setup → Normalized Handoff Record → Connected Accounts (canonical storage) → Social Icons Block (read only) → Renderer → Public Profile

---

## Relationship With Rendering Engine

The Rendering Engine:

* Consumes the Social Icons Render Object (content + resolved style)
* Preserves block order
* Composes it into the public profile

The Rendering Engine must not read raw blocks, select renderers, or mutate Render Objects. Renderer selection belongs to the Renderer Registry.

---

## Relationship With Block Renderer

The Block Renderer:

* Resolves connected accounts
* Produces a Render Object
* Attaches the Resolved Style it is given
* Handles missing accounts safely

The Block Renderer must not own social account data.

---

## V1 Exclusions

The Social Icons Block V1 does not support:

* Custom icons
* Custom icon uploads
* External URLs not linked to Connected Accounts
* Social analytics
* Follower counts
* Live social data
* Embedded feeds
* Social posts
* Social previews
* Per-icon styling controls

---

## Canonical Example

```json
{
  "id": "block_05",
  "account_id": "account_01",
  "type": "social_icons",
  "sort_order": 20,
  "content": {
    "accounts": [
      {
        "connected_account_id": "acc_instagram",
        "sort_order": 1
      },
      {
        "connected_account_id": "acc_linkedin",
        "sort_order": 2
      }
    ]
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
Social Icons Block
=
Collection Reference Block
```

It does not own social account data.

It owns only:

```text
selection
ordering
```

while the user-provided social account records, persisted in Connected Accounts, remain the canonical reference.
