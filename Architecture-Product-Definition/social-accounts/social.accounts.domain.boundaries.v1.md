# Social Accounts Domain Boundaries V1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the architectural boundaries of the Social Accounts domain.

Its purpose is to clearly define what this domain owns, what it intentionally ignores, and how it interacts with the rest of the Minime architecture.

The goal is to prevent responsibility overlap between domains and keep Social Accounts lightweight, predictable, and easy to maintain.

---

# Domain Responsibility

The Social Accounts domain owns only one responsibility:

> Collecting, normalizing, generating, and delivering canonical handoff records for user-provided social account links.

Everything inside this domain exists to support that single responsibility.

The Social Accounts domain is a processing domain. It does not maintain canonical storage. Connected Accounts is the canonical storage location for user social link records.

---

# Domain Inputs

The domain accepts user-provided social account identifiers.

Supported inputs include:

* Username
* Handle
* Public profile URL

Example:

```text
hmn

@hmn

https://instagram.com/hmn

https://x.com/hmn

https://linkedin.com/in/hmn
```

The domain accepts user input exactly as entered.

---

# Domain Outputs

The domain produces Normalized Social Account Handoff Records.

Each handoff record contains:

* account_id (ownership reference)
* Platform
* Normalized Identifier
* Generated Public URL
* Display Order
* Source Mode (Smart or Manual)

The handoff record is delivered to Connected Accounts for canonical storage.

Connected Accounts creates the canonical Connected Account record from the handoff record.

---

# What This Domain Owns

The Social Accounts domain owns:

* Smart Mode
* Manual Mode
* Input Normalization
* Username Extraction
* Platform Formatting Rules
* Public URL Generation
* Normalized Handoff Record Production
* Handoff Delivery to Connected Accounts

The Social Accounts domain does not own:

* Canonical storage of social link records
* Post-handoff ownership of social link data

---

# What This Domain Does Not Own

The domain intentionally does not own:

## Authentication

Handled by:

```text
Account
```

---

## Account Ownership

Handled by:

```text
OAuth Verification
```

(if enabled)

---

## Account Discovery

No discovery logic exists in V1.

---

## Username Search

No platform search exists.

---

## Platform Requests

No HTTP requests are sent to any social platform.

---

## Platform APIs

No platform API integrations exist.

---

## Web Scraping

Never performed.

---

## Browser Automation

Never performed.

---

## Search Engines

Never queried.

---

## Username Availability

Not supported.

---

## Account Existence

Never verified.

---

## Profile Reading

Not supported.

---

## AI Analysis

Outside this domain.

---

## Followers

Outside this domain.

---

## Statistics

Outside this domain.

---

## Synchronization

Outside this domain.

---

## Import

Outside this domain.

---

## Export

Outside this domain.

---

## Preview

Not supported.

---

## Drafts

Not supported.

---

## Publish Workflow

Not supported.

Saving immediately updates the user's public profile according to Minime's publishing model.

---

# Smart Mode Boundary

Smart Mode is frequently misunderstood.

Its responsibility is only:

```text
Input

↓

Normalize

↓

Generate URLs

↓

Save
```

It never performs:

```text
Search

↓

Discovery

↓

Verification

↓

Platform Communication
```

The word "Smart" refers only to reducing user typing.

It never refers to discovering accounts.

---

# Manual Mode Boundary

Manual Mode allows users to configure every platform individually.

It shares exactly the same normalization pipeline as Smart Mode.

The only difference is the source of the identifier.

---

# Platform Rules Boundary

Platform rules define only formatting behavior.

Examples include:

* URL templates
* Username formatting
* Handle prefixes
* Supported identifier patterns

Platform rules never define:

* API endpoints
* Search behavior
* Platform communication
* Verification logic

---

# OAuth Boundary

OAuth belongs to the Account domain.

Its responsibility is:

* ownership verification
* official platform authorization
* optional synchronization
* future platform integrations

OAuth is never required to create or save social accounts.

---

# Connected Accounts Boundary

Connected Accounts is the canonical storage location for user social link records.

The Social Accounts domain delivers handoff records to Connected Accounts.

Connected Accounts creates and owns the canonical records.

The Social Accounts domain does not read from Connected Accounts after handoff.

---

# Public Profile Boundary

The public profile surface (produced by Rendering) consumes social account data from Connected Accounts.

It does not modify Connected Accounts records.

It renders the canonical records as public links.

---

# Analytics Boundary

Analytics consumes click events generated by social account links.

It never modifies social account data.

---

# Rendering Boundary

Rendering receives social link data from Connected Accounts, not directly from the Social Accounts domain.

Rendering never attempts to repair or normalize account identifiers.

Rendering never regenerates URLs.

---

# Core Design Rules

## Rule 1

The Social Accounts domain trusts user input.

---

## Rule 2

The Social Accounts domain improves formatting.

It never changes user intent.

---

## Rule 3

The Social Accounts domain generates URLs.

It never validates destinations.

---

## Rule 4

The Social Accounts domain is completely independent from external social media services.

---

## Rule 5

Removing internet access must not affect the behavior of this domain.

Every operation inside this domain must continue functioning without communicating with any third-party platform.

---

# Canonical Boundary Statement

The Social Accounts domain is a pure data normalization and URL generation domain.

It has no dependency on social media platforms, platform APIs, search engines, scraping techniques, browser automation, or network-based account validation.

It does not maintain canonical storage.

Its sole responsibility is transforming user-provided identifiers into Normalized Social Account Handoff Records and delivering them to Connected Accounts for canonical storage.
