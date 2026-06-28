# Social Accounts Setup Specification V1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

The Social Accounts Setup domain is responsible for collecting the user's public social media account identifiers and generating normalized public profile URLs.

Its primary goal is to reduce user effort during onboarding while keeping the implementation simple, stable, and independent from third-party platform behavior.

This domain intentionally does **not** perform account discovery, ownership verification, profile analysis, or external platform validation.

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
* Ownership Verification
* OAuth Authentication
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

Smart Mode exists to minimize typing.

It assumes that the user commonly uses the same username across multiple platforms.

The user provides either:

* a common username

or

* a social profile URL

The system then:

1. extracts the identifier if necessary
2. normalizes the identifier
3. applies platform-specific URL templates
4. generates public profile URLs
5. saves the resulting social accounts

Smart Mode never performs platform requests.

It never attempts to determine whether the account actually exists.

Its responsibility ends after URL generation.

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

No external verification is performed.

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

# OAuth

OAuth is intentionally outside the scope of this domain.

If supported in future versions, OAuth exists only for:

* ownership verification
* platform integration
* synchronization
* advanced platform features

OAuth is never required to create social accounts.

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

It does not discover accounts, verify ownership, validate existence, communicate with external social media platforms, or maintain canonical storage of social link records.
