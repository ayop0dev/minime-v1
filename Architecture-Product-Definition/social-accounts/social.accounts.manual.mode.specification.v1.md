# Social Accounts Manual Mode Specification V1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

Manual Mode allows users to configure each social platform individually.

It exists for users who:

* use different usernames across platforms
* only use selected social platforms
* prefer entering accounts manually
* cannot use Smart Mode efficiently

Manual Mode prioritizes flexibility while following the exact same normalization pipeline used by Smart Mode.

---

# Design Goal

The primary objective of Manual Mode is to make individual social account entry as simple and consistent as possible.

The user should never be required to type complete profile URLs.

Instead, the user provides only the platform identifier while Minime generates the final public profile URL automatically.

---

# Core Philosophy

Manual Mode assumes:

> The user knows the correct identifier for each platform.

The system never attempts to discover, validate, or verify social accounts.

Its responsibility is limited to formatting and URL generation.

---

# User Experience

Each platform is configured independently.

Example:

```text
Instagram

@

______________
```

```text
TikTok

@

______________
```

```text
LinkedIn

/in/

______________
```

```text
YouTube

@

______________
```

The interface should guide the user toward the correct identifier format without requiring them to enter complete URLs.

---

# Supported Input

Each platform accepts:

* Username
* Handle
* Public Profile URL

If a full profile URL is provided, Manual Mode extracts the platform identifier automatically.

Example:

```text
Input

https://instagram.com/HMn/

↓

Identifier

HMn
```

The remaining URL components are discarded.

---

# Processing Pipeline

Every platform follows the same processing pipeline.

```text
Platform Selected

↓

User Input

↓

Normalize Input

↓

Apply Platform Rules

↓

Generate Public URL

↓

Save
```

Identifier extraction from a pasted URL is part of Normalize Input; it is not a separate stage in Manual Mode.

No additional processing occurs.

---

# Platform Independence

Each platform is completely independent.

Editing one platform never affects another.

Example:

```text
Instagram

hmn
```

does not modify

```text
TikTok

hmn_dev
```

Every social account is managed separately.

---

# Input Normalization

Before saving, the system normalizes the identifier.

Typical normalization includes:

* trimming whitespace
* removing unnecessary trailing slashes
* removing tracking parameters
* extracting usernames from supported URLs
* applying lowercase according to platform rules
* removing unnecessary platform prefixes

Normalization exists only to standardize formatting.

It never changes the intended identifier.

---

# URL Generation

After normalization, the system generates the official public profile URL for the selected platform.

Example:

```text
Instagram

hmn

↓

https://instagram.com/hmn
```

```text
TikTok

hmn

↓

https://tiktok.com/@hmn
```

```text
LinkedIn

hmn

↓

https://linkedin.com/in/hmn
```

---

# Save Behavior

Saving is immediate.

There is:

* no preview
* no draft
* no confirmation dialog
* no publish workflow

The normalized social account becomes part of the user's public profile according to Minime's publishing model.

---

# Platform Guidance

Every platform may provide contextual guidance to help users enter the correct identifier.

Examples include:

* leading @ symbol
* fixed URL prefixes
* placeholder examples
* platform-specific formatting hints

These elements exist only to improve usability.

They never perform validation against external platforms.

---

# What Manual Mode Does

Manual Mode:

* accepts platform-specific identifiers
* accepts supported profile URLs
* extracts identifiers
* normalizes formatting
* applies platform rules
* generates public URLs
* produces Normalized Social Account Handoff Records
* delivers handoff records to Connected Accounts

---

# What Manual Mode Never Does

Manual Mode never:

* checks account existence
* validates ownership
* performs discovery
* communicates with social platforms
* calls platform APIs
* performs web scraping
* opens profile pages
* performs browser automation
* verifies usernames
* suggests replacements
* modifies user intent

---

# User Responsibility

The user is responsible for entering the intended identifier.

Example:

```text
Input

hmn
```

If the intended username was actually:

```text
hms
```

The system preserves:

```text
hmn
```

No correction is attempted.

---

# System Responsibility

The system is responsible only for formatting.

Example:

```text
HMN
```

↓

```text
hmn
```

Example:

```text
https://linkedin.com/in/HMN/
```

↓

```text
hmn
```

Formatting may change.

Meaning must remain unchanged.

---

# Relationship with Smart Mode

Smart Mode and Manual Mode share:

* the same normalization engine
* the same platform rules
* the same URL generation logic
* the same handoff record model
* the same handoff delivery to Connected Accounts

The only difference is how the identifier is collected.

---

# Design Principles

## Platform First

Each social platform is configured independently.

---

## Reduce Typing

Users provide identifiers, not complete URLs.

---

## Normalize, Don't Correct

Formatting improves.

Meaning never changes.

---

## Trust the User

The user owns the identifier.

The system simply stores it in a standardized format.

---

## Zero External Dependencies

Manual Mode operates entirely without communicating with any social platform.

---

# Canonical Statement

Manual Mode provides platform-specific collection of user-provided social account identifiers.

It normalizes formatting, generates standardized public profile URLs, and delivers Normalized Social Account Handoff Records to Connected Accounts for canonical storage — without performing account discovery, account verification, or external platform communication.
