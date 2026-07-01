# Social Accounts Setup Specification V1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

The Social Accounts Setup domain is responsible for collecting the user's public social media account identifiers and generating normalized public profile URLs.

Its primary goal is to reduce user effort during onboarding while keeping the implementation simple, stable, and independent from third-party platform behavior.

Its responsibility is limited to canonical input processing: normalizing what the user provides and generating the corresponding public URL.

---

# Philosophy

Minime trusts the user.

If a user says:

> This is my Instagram username.

The system accepts that statement.

The responsibility of this domain is to normalize the provided identifier according to the selected platform rules and generate the correct public profile URL.

The correctness of the account itself remains the user's responsibility.

---

# Objectives

The Social Accounts Setup domain exists to:

* Collect social media accounts.
* Reduce typing.
* Normalize user input.
* Generate valid public profile URLs.
* Store normalized social account records.

Nothing more.

---

# Non Objectives

This domain never performs:

* Account Discovery
* Username Search
* Platform Search
* Ownership Confirmation
* API Communication
* Web Scraping
* Browser Automation
* Search Engine Queries
* Account Existence Validation
* Followers Retrieval
* Profile Reading
* AI Analysis
* Confidence Scoring
* URL Preview
* Draft Creation
* Publish Confirmation

These responsibilities belong to other domains if introduced in future versions.

---

# User Experience

The user can choose one of two setup methods.

```text
Social Accounts Setup

├── Smart Mode
└── Manual Mode
```

Both methods ultimately produce the same normalized social account records.

---

# Smart Mode

Smart Mode is defined canonically in `social.accounts.smart.mode.specification.v1.md`. That document is the single source of truth for Smart Mode's responsibilities and constraints.

Within Social Accounts Setup, choosing Smart Mode only changes how the identifier is collected. It produces a Normalized Social Account Handoff Record through the same pipeline used by Manual Mode.

---

# Manual Mode

Manual Mode allows users to configure every platform individually.

Each platform provides its own input guidance based on platform conventions.

Examples:

* @username
* username
* /in/username
* @handle

The system normalizes the input according to platform rules before saving.

---

# Input Normalization

Before saving any account, the system normalizes the user input.

Typical normalization includes:

* trimming whitespace
* extracting usernames from supported URLs
* removing tracking query parameters
* removing unnecessary trailing slashes
* applying lowercase where appropriate
* removing platform prefixes when applicable
* applying platform-specific formatting rules

Normalization improves consistency.

Normalization never changes the user's intended identifier.

---

# URL Generation

After normalization, the system generates the final public profile URL using the official URL template for each platform.

Example:

```text
Identifier

↓

Normalized Identifier

↓

Platform URL Template

↓

Generated Public URL
```

The transformation occurs entirely from the normalized identifier and the platform's URL template.

---

# Saving

After URL generation, the Social Accounts domain produces a Normalized Social Account Handoff Record.

The handoff record is immediately delivered to Connected Accounts for canonical storage.

Connected Accounts creates the canonical Connected Account record.

There are no drafts.

There is no preview mode.

There is no publish confirmation.

The Connected Account immediately becomes part of the user's public profile according to the normal publishing behavior of Minime.

The Social Accounts domain does not maintain canonical storage.

Connected Accounts is the canonical storage location for user social link records.

---

# Domain Responsibilities

This domain is responsible for:

* Social account collection
* Smart Mode
* Manual Mode
* Input normalization
* Platform formatting rules
* Public URL generation
* Normalized handoff record production
* Handoff delivery to Connected Accounts

This domain is not responsible for:

* Canonical storage of social link records
* Post-handoff ownership of social link data

---

# Domain Boundaries

This domain does not know:

* whether a username exists
* whether a profile is public
* whether the account belongs to the current user
* whether a profile contains content
* whether a platform is reachable

Those concerns intentionally remain outside this domain.

---

# Design Principles

The Social Accounts Setup domain follows these principles.

## Trust the User

The user is the source of truth for their own social accounts.

---

## Normalize, Don't Guess

The system corrects formatting.

The system never guesses user intent.

---

## Generate, Don't Discover

The system generates public URLs.

It never searches for accounts.

---

## Keep Platform Independent

No implementation depends on platform APIs, search engines, scraping, or browser automation.

---

## Minimize Maintenance

The architecture favors long-term stability over aggressive automation.

---

# Cross-Domain Handoff Contract

## Downstream Consumer

Connected Accounts

## What Is Delivered

The Social Accounts domain produces a Normalized Social Account Handoff Record after each successful collection, normalization, and URL generation cycle.

## Handoff Record Fields

| Field | Description |
|---|---|
| account_id | Reference to the owning Account |
| platform | Platform identifier |
| identifier | Normalized username |
| public_url | Generated canonical URL |
| display_order | Ordering intent |
| source_mode | Collection method (Smart or Manual) |

## On Delivery

Connected Accounts receives the handoff record and creates a canonical Connected Account record.

Connected Accounts assigns `connected_account_id` upon creation.

Connected Accounts is thereafter the canonical owner of the social link data.

## Social Accounts Responsibilities at Handoff

The Social Accounts domain does not store the handoff record independently.

The Social Accounts domain does not write to Connected Accounts directly after handoff.

The Social Accounts domain does not read from Connected Accounts.

## Validation at Handoff

The Social Accounts normalization engine validates format before producing the handoff record.

Records that fail normalization are rejected before the handoff record is produced.

Connected Accounts does not re-validate records received through Social Accounts Setup.

---

# Canonical Statement

Social Accounts Setup is responsible for collecting, normalizing, and producing normalized handoff records for user-provided social account identifiers.

It generates standardized public profile URLs using platform-specific rules and delivers the resulting handoff records to Connected Accounts for canonical storage.

Its responsibility begins and ends with canonical input processing: it trusts the identifier the user provides, transforms it into a standardized format, and generates the corresponding public URL.
