# Profile Block Reference Specification V1

## Status

Approved

---

# Purpose

This document defines how Profile Content references Blocks in Minime V1.

It answers:

* How Blocks are connected to Profile Content
* How Block ownership works
* How Block ordering works
* How Profile Content retrieves Blocks
* What data Profile Content stores about Blocks
* What data Profile Content does not store about Blocks

This document does not define:

* Block implementation
* Block validation
* Block rendering
* Theme behavior
* Style resolution

Those responsibilities belong to their own systems.

---

# Definition

Profile owns the collection of Blocks displayed on the public profile.

Blocks represent the content units shown on a profile.

Examples:

```text
Avatar
Name
Bio
Button
Image
Social Icons
Divider
Title
Textbox
```

Profile Content does not own block implementation.

Profile owns block references.

---

# Ownership Model

Relationship:

```text
Account
↓
Profile Content
↓
Block References
↓
Blocks
```

Every Block belongs to exactly one Account.

Every Block may belong to only one Profile Content collection.

Blocks cannot be shared between Accounts.

---

# Reference Strategy

Profile Content references Blocks using Block IDs.

Example:

```text
blk_001
blk_002
blk_003
blk_004
```

Profile Content does not duplicate Block content.

Profile Content only stores references.

---

# Why References Exist

The Block System owns:

```text
Block Identity
Block Type
Block Content
Block Settings
Block Validation
```

Profile owns:

```text
Block Collection
Block Ordering
Block Composition
```

This keeps responsibilities separated.

---

# Block Collection

Profile Content does not store a `blocks[]` or `block_ids[]` field. The block collection for an Account is the query result `SELECT * FROM Block WHERE account_id = :account_id AND deleted_at IS NULL ORDER BY sort_order` — see "Canonical Composition Model" below.

Example (the resulting collection, not a stored field):

```text
blk_avatar,
blk_name,
blk_bio,
blk_button_1,
blk_button_2
```

`account_id` on each `Block` row is what defines which Blocks belong to the Account — not a reference list stored on Profile Content.

---

# Ordering Ownership

Profile owns block ordering.

The Block itself does not decide where it appears.

Example:

```text
Avatar
↓
Name
↓
Bio
↓
Button
↓
Button
```

This order belongs to Profile.

Not to the individual Blocks.

---

# Ordering Model

Ordering may be represented by:

```text
sort_order
```

inside each Block

or

```text
ordered block references
```

inside Profile Content.

For V1 the source of truth should remain:

```text
Block.sort_order
```

because this is already defined by the Block System.

Profile Content consumes the final ordering.

It does not create a second ordering model.

---

# Block Discovery

To retrieve profile blocks:

```text
Profile Content
↓
Get Account Blocks
↓
Sort
↓
Return Collection
```

Profile Content does not render the result.

It only provides the collection.

---

# Block Rendering Eligibility

Every stored Block is eligible for rendering.

A Block either exists or it does not exist. There is no hidden, archived, or inactive block state.

Example:

```text
Avatar
Name
Bio
Button
```

Returned collection:

```text
Avatar
Name
Bio
Button
```

---

# Identity Blocks

Identity Blocks are part of the block collection.

Examples:

```text
Avatar
Name
Bio
```

Even though they represent profile identity information, they are still Blocks.

They follow the same ownership model.

---

# Content Blocks

Content Blocks are also part of the block collection.

Examples:

```text
Button
Image
Social Icons
Divider
Title
Textbox
```

They follow the same reference model.

---

# Block Creation

Profile Content may receive new Block references when:

```text
Block Created
↓
Assigned To Account
↓
Added To Collection
```

The Block System creates the Block.

Profile Content includes it in the collection.

---

# Block Updates

Block updates do not change the Block reference.

Example:

```text
Block ID
=
blk_001
```

User changes content:

```text
Old Content
↓
New Content
```

Reference remains:

```text
blk_001
```

Profile Content remains unchanged.

---

# Block Reordering

Reordering changes composition.

Not ownership.

Example:

Before:

```text
Avatar
Name
Bio
Button
```

After:

```text
Avatar
Name
Button
Bio
```

The same Block references remain.

Only ordering changes.

---

# Block Removal

Removing a Block means:

```text
Remove Reference
↓
Delete Block
```

Profile Content must no longer include deleted blocks in profile composition.

---

# Rendering Relationship

Profile Content does not render Blocks.

Flow:

```text
Profile Content
↓
Block Collection
↓
Block Renderer
↓
Render Object
↓
Rendering Engine
↓
Public Output
```

Profile Content stops at block composition.

---

# Data Profile Content Must Not Store

Profile Content must not duplicate:

```text
Block Content
Block Settings
Block Validation Rules
Block Rendering Data
Theme Data
Style Data
```

These belong elsewhere.

---

# Data Profile Content May Store

Profile Content stores only its own canonical fields (`display_name`, `bio`, `avatar_storage_key`, `contact`, `appearance_config` — see `implementation/03-canonical-data-model.md` — ProfileContent). It stores no Block references, collection metadata, or composition metadata of any kind — composition is always derived by querying `Block` directly (see "Canonical Composition Model").

---

# Relationship With Block System

Block System owns:

```text
What A Block Is
Block Rules
Block Lifecycle
Block Validation
```

Profile owns:

```text
Which Blocks Belong To The Account
Which Order They Appear In
```

---

# Relationship With Rendering Engine

The blocks are transformed into Render Objects by the Block Renderer.

The Rendering Engine receives:

```text
Render Objects
```

The Rendering Engine composes the public profile from Render Objects.

Profile Content remains read-only during rendering.

---

# Unsupported V1 Features

The Block Reference model does not support:

```text
Nested Blocks
Block Groups
Shared Blocks
Reusable Blocks
Cross-Profile Blocks
Conditional Blocks
Scheduled Blocks
AI Generated Block References
```

Each Block belongs to one Account only.

---

# Canonical Composition Model

```ts
type ProfileContent = {
  account_id: string;
  // no block_ids or blocks[] field — see "Ordering Model" above: Block.sort_order
  // is the single source of truth, and ProfileContent does not duplicate it into
  // a second ordering array. Composition is a query, not a stored list:
  //   SELECT * FROM Block WHERE account_id = :account_id AND deleted_at IS NULL
  //   ORDER BY sort_order
};
```

An earlier draft of this document modeled composition as a stored `block_ids: string[]` array on `ProfileContent`. That was a second ordering model and directly contradicted "Ordering Model" above (`Block.sort_order` is the source of truth; "It does not create a second ordering model"). It also never matched `implementation/03-canonical-data-model.md`'s `ProfileContent` entity, which has no `block_ids` or `blocks` field. This section is corrected to match both: composition and ordering come from querying `Block` directly, not from a field stored on `ProfileContent`.

The actual Block data remains inside the Block System.

---

# Canonical Ownership Rule

```text
Account
↓
Profile Content
↓
Block References
↓
Blocks
```

The Account owns the Profile Content.

Profile owns the block collection.

The Block System owns the Blocks themselves.

---

# System Invariant

For Minime V1:

```text
Account
↓
Profile Content
↓
Block Collection
↓
Block Renderer
↓
Render Object
↓
Rendering Engine
↓
Public Profile
```

Profile Content is responsible for composition.

The Block System is responsible for block ownership and behavior.

The Rendering Engine is responsible for visual output.
