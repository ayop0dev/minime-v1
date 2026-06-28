# Profile Content Specification V1

## Status

Approved

---

# Purpose

> Profile Content is part of the Profile domain. The Profile domain owns the profile as one business concept — its profile fields, its blocks, and its design (appearance). Profile fields, blocks, and design are implementation concerns within Profile, not separate business ownership boundaries. This document is the primary specification of the Profile domain.

This document defines the content model used by Minime profiles.

Profile Content represents the information a user chooses to publish through their Minime profile.

This specification is responsible for:

* Profile Information
* Content Blocks
* Contact Information

This specification is not responsible for:

* Themes
* Layouts
* Rendering
* Design Decisions
* Authentication
* Connected Accounts
* Social Accounts

---

# Core Principle

## Content Only

Profile Content owns:

```text
What The User Publishes
```

Appearance owns:

```text
Visual Theme Selection And Customization
```

Rendering owns:

```text
How Content Is Assembled And Delivered
```

---

# Separation Of Responsibilities

## Profile Content

Responsible for:

* Display Name
* Avatar
* Bio
* Contact Information
* Blocks
* Block Data

---

## Appearance system

Responsible for:

* Visual Theme Selection
* Theme Customization
* Background Configuration
* Appearance State

---

## Rendering Domain

Responsible for:

* Design Tokens
* Layout Rules
* Rendering Rules
* Responsive Behavior
* Visual Styling

---

# Conceptual Architecture

> [Conceptual Architecture View — Domain Ownership]

Profile Content is one of three domains that together define what a public profile is and how it is presented.

```text
Profile Content          ← owns: what the user publishes
Appearance               ← owns: visual theme selection and customization
Rendering                ← owns: how content is assembled and delivered
```

These are three distinct domains with distinct responsibilities. They do not share ownership.

---

# Persistence View

> [Persistence View — Storage Structure]

Domain boundary does not equal storage boundary.

Profile Content and Appearance State are stored together as a single persisted record per account.

```text
Profile Content Record (per account_id)
├─ account_id
├─ display_name
├─ avatar
├─ bio
├─ contact_information
├─ blocks []
└─ appearance_state
   ├─ selected_theme_id
   └─ theme_customization_values
```

Appearance State does not require its own separate persisted entity. It passes zero conditions of the Persistence Test (no multiplicity, no independent lifecycle, no independent reference from other entities). It is stored as a section within this record.

---

# Canonical Notation

When other specifications reference Profile Content fields, they must use the `ProfileContent.x` notation.

```text
ProfileContent.display_name
ProfileContent.avatar
ProfileContent.bio
```

This notation is canonical across all domain specifications, block specifications, and rendering specifications.

The notation `Profile.display_name`, `Profile.avatar`, or `Profile.bio` must not be used.

`Profile` is not a persisted entity in V1. See Canonical Entities Map §31.

---

# Current State Only

Profile Content stores only the current version of content.

There is no:

* Version History
* Revision History
* Draft System
* Restore Functionality
* Rollback Functionality

---

# Last Write Wins

Profile Content follows:

```text
Last Write Wins
```

New values replace previous values.

Previous values are discarded.

---

# Profile Model

## Required Fields

Every profile contains:

```text
account_id
display_name
avatar
bio
contact_information
```

---

## Example

```json
{
  "account_id": "usr_01JXXXX",
  "display_name": "Ahmed Salem",
  "avatar": "avatar.jpg",
  "bio": "Building products and businesses.",
  "contact_information": {
    "email": "hello@example.com",
    "phone": "+201234567890",
    "whatsapp": "+201234567890",
    "location": "Cairo, Egypt"
  }
}
```

---

# Display Name

## Purpose

Display Name is the public name shown on the profile.

Example:

```text
Ahmed Salem
```

---

## Requirement

Display Name is required.

Every profile must contain a Display Name.

---

## Uniqueness

Display Names are not unique.

Example:

```text
Ahmed Salem
Ahmed Salem
Ahmed Salem
```

may all exist.

---

# Avatar

## Purpose

Avatar represents the profile image.

---

## Ownership

Avatar is Profile Content.

Avatar is not a presentation setting.

---

## Requirement

Avatar is optional.

A profile may exist without an avatar.

---

# Bio

## Purpose

Bio is the primary profile description.

---

## Requirement

Bio is optional.

A profile may exist without a bio.

---

## Maximum Length

Bio maximum length:

```text
1000 Characters
```

---

# Contact Information

## Purpose

Contact Information provides public communication channels.

---

## Supported Fields

V1 supports:

```text
email
phone
whatsapp
location
```

---

## Requirement

All contact fields are optional.

A profile may contain:

```text
0
```

contact methods.

---

## Visibility

If a field exists:

```text
Display It
```

If a field does not exist:

```text
Do Not Display It
```

---

# Blocks

## Purpose

Blocks are the smallest content unit in Minime.

---

## Ownership

Blocks belong to:

```text
Account
```

---

## Block Structure

Every block contains:

```text
block_id
account_id
type
data
sort_order
created_at
updated_at
```

---

## Example

```json
{
  "block_id": "blk_01JXXXX",
  "account_id": "usr_01JXXXX",
  "type": "button",
  "sort_order": 10
}
```

---

# Supported Block Types

V1 supports:

```text
image
name
bio
button
social_icons
divider
text
```

Block rendering is defined separately.

---

# Button Ownership

Buttons are not profile fields.

Buttons are blocks.

Structure:

```text
Profile
↓
Button Block
```

---

# Block Ordering

## User Controlled Ordering

Blocks support:

```text
Drag & Drop
```

ordering.

---

## Display Order

Blocks are displayed according to:

```text
sort_order
```

ascending.

---

# Empty Profiles

## Allowed

Profiles may exist with:

* No Avatar
* No Bio
* No Contact Information
* No Buttons

---

## Example

Immediately after registration:

```text
Display Name Only
```

is a valid profile.

---

# Publishing Model

## No Draft Workflow

Profile Content does not support drafts.

Changes are saved immediately, with no separate publish step. Saved changes then propagate to the visitor-facing profile as near-live updates, with a maximum cache delay of 60 seconds.

Flow:

```text
Edit
↓
Save
↓
Near-Live Public Update (≤ 60s cache delay)
```

---

## Unsupported

V1 does not support:

* Drafts
* Scheduled Publishing
* Approval Workflows
* Version History
* Rollback

---

# Out Of Scope

This specification does not define:

* Themes
* Rendering
* Layout Rules
* Design Tokens
* Responsive Behavior
* Analytics
* Profile Import
* Social Accounts

These belong to separate systems.

---

# Related Documents

```text
minime.account.management.system.specification.v1.md
connected.accounts.specification.v1.md

theme.catalog.specification.v1.md
theme.definition.specification.v1.md
theme.selection.specification.v1.md
theme.customization.specification.v1.md
theme.constraints.specification.v1.md

layout.system.specification.v1.md
rendering.engine.specification.v1.md
rendering/rendering.system.specification.v1.md
```
