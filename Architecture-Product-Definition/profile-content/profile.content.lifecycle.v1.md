# Profile Content Lifecycle Specification V1

## Status

Approved

---

# Purpose

This document defines the lifecycle of Profile Content in Minime V1.

It describes:

* How Profile Content is created
* How Profile Content evolves
* How Profile Content is updated
* How Profile Content is rendered
* How Profile Content is removed

This document does not define:

* Account creation
* Authentication
* Username ownership
* Theme behavior
* Rendering implementation
* Analytics

---

# Definition

Profile Content represents the public-facing content owned by an Account.

Examples:

```text
Display Name
Avatar
Bio
Contact Information
Blocks
Block Order
```

Profile Content is not a separate entity.

Profile Content belongs directly to an Account.

---

# Lifecycle Overview

```text
Account Created
↓
Profile Content Created
↓
Content Updated
↓
Content Rendered
↓
Content Updated Again
↓
Content Rendered Again
↓
Account Removed
↓
Profile Content Removed
```

Profile Content always follows the lifecycle of its owning Account.

---

# Creation

## Creation Trigger

Profile Content is created automatically when a Minime Account is successfully created.

Flow:

```text
Username Claimed
↓
Account Created
↓
Profile Content Created
```

The user never manually creates Profile Content.

Profile Content always exists for every Account.

---

## Initial State

Immediately after creation:

```text
Display Name = the claimed username (set once, at creation, by the registration transaction; never empty)
Avatar = Empty
Bio = Empty
Blocks = Empty
```

Display Name is the one field that is never empty, from creation onward — see `profile.content.specification.v1.md` and `implementation/03-canonical-data-model.md`. All other fields may remain empty.

The profile may be incomplete.

Incomplete content is valid, except for Display Name, which is always populated.

---

# First Content Population

Content may be populated through:

```text
Manual Editing
Social Accounts Setup
Future Import Systems
```

In V1:

```text
Manual Editing
```

is the primary mechanism.

---

# Updating Content

## Update Strategy

Profile Content follows:

```text
Last Write Wins
```

Flow:

```text
Current Value
↓
User Saves New Value
↓
Old Value Replaced
```

The latest saved value becomes active immediately.

---

## No Draft System

V1 does not support:

```text
Drafts
Pending Changes
Review Queues
Approval Flows
```

Example:

```text
Edit Bio
↓
Save
↓
Bio Updated
```

There is no intermediate state.

---

## No Version History

Profile Content stores only the current state.

V1 does not support:

```text
Version History
Restore Points
Rollback
Revision Tracking
```

Example:

```text
Bio A
↓
Bio B
```

Bio A is discarded.

---

# Block Lifecycle

Blocks are part of Profile Content.

Each Block may be:

```text
Created
Updated
Reordered
Deleted
```

according to the Block System rules.

Profile owns the collection of blocks.

The Block System owns block-specific lifecycle behavior.

---

# Reordering

Users may reorder blocks.

Flow:

```text
Current Order
↓
Drag And Drop
↓
New Order Saved
```

The latest ordering becomes active immediately.

There is no publish step.

---

# Rendering Relationship

Profile Content does not render itself.

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
Public Profile
```

Profile Content provides data.

Rendering systems provide visual output.

---

# Profile Availability Relationship

Profile Content itself does not have draft or published states.

There is no:

```text
Draft
Published
Archived
Private Profile Content
```

at the content level.

A block either exists or it does not exist. There is no hidden block state.

Whether a profile is publicly reachable depends on:

```text
Public Profile System
Account Status
```

depending on context.

---

# Empty Content

Empty content is allowed.

Examples:

```text
No Avatar
No Bio
No Buttons
No Social Icons
```

The profile remains valid.

The renderer decides how to display missing content.

---

# Content Removal

## Field Removal

Users may clear content.

Example:

```text
Bio Exists
↓
Remove Bio
↓
Bio Empty
```

The content field remains part of the profile structure.

Only its value changes.

---

## Block Removal

Blocks may be permanently removed.

Flow:

```text
Existing Block
↓
Delete
↓
Block Removed
```

The Block no longer exists.

V1 does not support:

```text
Trash
Archive
Restore
Undo Delete
```

---

# Account Relationship

Profile Content cannot exist without an Account.

Invalid:

```text
Profile Content
Without
Account
```

Valid:

```text
Account
↓
Profile Content
```

Account is the owner.

Profile Content is dependent.

---

# Account Deletion

If the Account is removed:

```text
Account Removed
↓
Profile Content Removed
```

Profile Content must never outlive its owning Account.

---

# Unsupported Lifecycle States

Profile Content does not support:

```text
Draft
Pending
Review
Published
Archived
Imported
Locked
Frozen
Scheduled
```

The content either exists or does not exist.

The latest saved state is always the active state.

---

# Lifecycle Principles

## Always Exists

Every Account always has Profile Content.

---

## Current State Only

Only the current content state is stored.

---

## Immediate Updates

Changes become active immediately after saving.

---

## No Publishing Workflow

Saving content is sufficient.

No additional publish action exists.

---

## Account Owned

Profile Content belongs directly to the Account.

It is not an independent entity.

---

# System Invariant

For Minime V1:

```text
Account
↓
Profile Content
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

Profile Content always remains an Account-owned content container.

It never becomes a separate identity, profile, or aggregate root.
