# Connected Accounts Specification V1

## Status

Approved

---

# Purpose

This document defines the Connected Account model used by Minime.

Connected Accounts represent external platform accounts that a user chooses to associate with their Minime profile.

Connected Accounts is the canonical storage location for user social link records.

Connected Accounts are lightweight account references.

They are not:

* Ownership verification records
* Profile import records
* Analytics records
* Identity records

---

# Scope

This specification defines:

* Connected Account model
* Connected Account lifecycle
* Creation rules
* Validation rules
* Editing rules
* Ordering rules
* Removal rules

This specification does not define:

* Social Accounts Setup
* Profile Import
* Analytics
* Ownership Verification
* Identity Verification

---

# Core Principles

## User Is The Source Of Truth

Minime never decides which accounts belong to a user.

The user decides:

* Which accounts to add
* Which accounts to remove
* Which accounts to update
* How accounts are ordered

The system only validates the format of each connected account record. It never checks whether the account exists on the external platform.

---

## Lightweight Entity

Connected Accounts are intentionally lightweight.

The purpose of a Connected Account is:

```text
Platform
+
Username
↓
Canonical URL
```

Nothing more.

---

## No Status Model

Connected Accounts do not have statuses.

There is no:

* Pending
* Active
* Inactive
* Imported
* Verified
* Hidden
* Archived

A Connected Account either exists or does not exist.

---

## Current State Only

Connected Accounts store only the current state.

There is no:

* Version history
* Draft system
* Restore functionality
* Rollback functionality

---

# Connected Account Model

## Required Fields

Every Connected Account contains:

```text
connected_account_id
account_id
platform
username
url
sort_order
created_at
updated_at
```

---

## Example

```json
{
  "connected_account_id": "acc_01JXXXX",
  "account_id": "usr_01JXXXX",
  "platform": "instagram",
  "username": "ahmedofficial",
  "url": "https://instagram.com/ahmedofficial",
  "sort_order": 10
}
```

---

# Field Definitions

## connected_account_id

Connected Account identifier.

Identifies the connected record itself.

Requirements:

```text
Globally Unique
Immutable
Permanent
```

Example:

```text
acc_01JXXXX
```

The value never changes.

---

## account_id

Owner of the Connected Account.

Identifies the owning Minime Account.

References:

```text
Account
```

---

## platform

Platform identifier.

Examples:

```text
instagram
tiktok
youtube
linkedin
x
threads
```

Platform values must come from the Social Accounts Platform Registry.

---

## username

Platform username.

Example:

```text
ahmedofficial
```

Stored using the platform's canonical format.

---

## url

Canonical account URL.

Example:

```text
https://instagram.com/ahmedofficial
```

URL is:

```text
Derived Data
```

URL is not user editable.

The system generates the URL using the platform rules.

---

## sort_order

User-controlled display order.

Used for:

```text
Drag And Drop Ordering
```

Lower values appear first.

---

# URL Ownership

## Canonical URL Generation

Connected Accounts never store arbitrary URLs.

The URL is always generated from:

```text
platform
+
username
```

using the platform rules.

Example:

```text
instagram
+
ahmedofficial
↓
https://instagram.com/ahmedofficial
```

---

## User Input

Users never provide URLs.

Users provide:

```text
platform
+
username
```

only.

---

# Creation

## Social Accounts Setup

Connected Accounts may be created from Social Accounts Setup.

Flow:

```text
Normalized Social Account Record
↓
Save
↓
Create Connected Account
```

Account creation occurs immediately.

No intermediate state exists.

---

## Manual Addition

Connected Accounts may be created manually.

Flow:

```text
Platform
+
Username
↓
Validation
↓
Create Connected Account
```

---

# Validation

## Validation Requirement

All Connected Accounts must pass validation before creation.

Validation uses the platform rules.

---

## Successful Validation

If the format is valid:

```text
Create Connected Account
```

---

## Failed Validation

If the format is invalid:

```text
Creation Rejected
```

The account is not stored.

---

## Validation Authority

Validation rules belong to the Social Accounts Platform Registry.

The Social Accounts Platform Registry defines:

* Supported platforms
* Username format rules
* URL templates

Account Management reads platform rules from the Social Accounts Platform Registry.

Account Management must not implement platform-specific validation logic independently.

For records received through Social Accounts Setup, the Social Accounts normalization engine has already validated format before the handoff record is produced. Connected Accounts creation from Social Accounts Setup receives pre-validated records.

For records created through Manual Addition, Connected Accounts validates username format against the Social Accounts Platform Registry rules before creation.

---

# Duplicate Prevention

Duplicate Connected Accounts are not allowed.

Example:

```text
Instagram
ahmedofficial
```

cannot exist twice for the same user.

---

# Editing

## Editable Fields

Users may edit:

```text
username
```

only.

---

## URL Editing

Direct URL editing is not supported.

The URL is regenerated automatically after successful validation.

---

## Update Validation

All username updates require validation before saving.

Flow:

```text
Existing Account
↓
New Username
↓
Validation
↓
Save
```

---

## Failed Validation

If the new username cannot be validated:

```text
Update Rejected
```

The existing Connected Account remains unchanged.

---

## Connected Account Identity

Editing does not create a new connected record.

Example:

```text
connected_account_id = acc_01JXXXX
```

remains unchanged.

---

# Platform Changes

Platform changes are not supported.

Example:

```text
Instagram
↓
TikTok
```

is not an update.

It is a different account.

Flow:

```text
Delete Existing Account
↓
Create New Account
```

---

# Ordering

## User Controlled Ordering

Connected Accounts support manual ordering.

Users may reorder accounts using:

```text
Drag And Drop
```

---

## Persistence

Ordering changes are saved immediately.

The latest ordering becomes the active ordering.

---

## Display Order

Connected Accounts should be displayed according to:

```text
sort_order
```

ascending.

---

# Removal

## Removal Method

Connected Accounts use:

```text
Hard Delete
```

---

## Flow

```text
Remove Account
↓
Delete Connected Account
```

The account is permanently removed.

---

## Unsupported

V1 does not support:

* Trash
* Archive
* Restore
* Undo Delete

---

# Visibility

## Visibility Model

Connected Accounts do not have visibility states.

There is no:

```text
visible
hidden
draft
private
```

If an account exists:

```text
It Exists
```

If it should not exist:

```text
Delete It
```

---

# Social Accounts Integration

## Social Accounts Relationship

Social Accounts Setup is a processing domain.

Connected Accounts is the canonical storage location for the records that Social Accounts Setup produces.

Social Accounts Setup delivers Normalized Social Account Handoff Records to Connected Accounts.

Connected Accounts creates canonical Connected Account records from those handoff records.

Social Accounts Setup does not maintain its own canonical storage.

---

## Existing Accounts

If Social Accounts Setup produces a record that already exists:

```text
Already Added
```

may be displayed to the user.

No duplicate account should be created.

---

# Cross-Domain Contracts

## Social Accounts Setup (Producer)

| Role | Domain |
|---|---|
| Producer | Social Accounts |
| Canonical Storage | Connected Accounts |

### What Is Delivered

Social Accounts Setup delivers a Normalized Social Account Handoff Record after each successful collection, normalization, and URL generation cycle.

### Field Mapping Received

| Handoff Record Field | Connected Account Field |
|---|---|
| account_id | account_id |
| platform | platform |
| identifier | username |
| public_url | url |
| display_order | sort_order |

### Ownership After Handoff

Connected Accounts owns the canonical record after creation.

Social Accounts does not retain canonical ownership of delivered records.

### Write Authority

Social Accounts delivers records to Connected Accounts through the handoff mechanism only.

Social Accounts must not modify Connected Account records after creation.

### Read Authority

Social Accounts has no read requirement against Connected Accounts.

---

## Social Icons Block (Consumer)

| Role | Domain |
|---|---|
| Canonical Storage | Connected Accounts |
| Consumer | Social Icons Block |

### What Is Consumed

The Social Icons Block renderer resolves `connected_account_id` references by reading from Connected Accounts.

The renderer reads:

* platform
* username
* url

### Ownership

Connected Accounts owns platform, username, and url.

The Social Icons Block owns account selection, ordering, and visibility within the block.

### Write Authority

The Social Icons Block must not write to Connected Accounts.

### Read Authority

The Social Icons Block renderer reads from Connected Accounts by resolving `connected_account_id`.

Changes to Connected Accounts records automatically affect any Social Icons Block that references them.

---

# Out Of Scope

Connected Accounts must not store:

* Display Name
* Avatar
* Bio
* Followers
* Following
* Verification Status
* Website
* Analytics
* Metrics
* Imported Profile Data

These belong to separate systems.

---

# Related Documents

```text
minime.account.management.system.specification.v1.md
../social-accounts/social.accounts.setup.specification.v1.md
../social-accounts/social.accounts.platform.rules.v1.md
```
