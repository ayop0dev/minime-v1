# Social Accounts Input Normalization Specification V1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines how Minime normalizes user-provided social account identifiers before they are stored.

Normalization exists to produce consistent, predictable, and standardized social account records regardless of how the user enters their information.

Normalization improves formatting.

It never changes user intent.

---

# Design Philosophy

The user is responsible for the identifier.

The system is responsible for the format.

Example:

```text
User

HMN
```

↓

```text
Stored

hmn
```

The meaning remains identical.

---

Example:

```text
User

hmn
```

The system never changes it into:

```text
hms
```

because doing so would change user intent.

---

# Normalization Pipeline

Every identifier follows the same pipeline.

```text
User Input

↓

Trim

↓

Extract Identifier

↓

Remove Platform Noise

↓

Apply Platform Rules

↓

Generate Normalized Identifier

↓

Store
```

Every Smart Mode and Manual Mode input uses this identical pipeline.

---

# Rule 1

## Trim Whitespace

Leading and trailing whitespace must always be removed.

Example:

```text
"   hmn   "
```

↓

```text
hmn
```

---

# Rule 2

## Remove Empty Lines

Line breaks are removed.

Example:

```text
hmn

```

↓

```text
hmn
```

---

# Rule 3

## Extract Identifier From Supported URLs

If a supported public profile URL is detected, the identifier is extracted.

Example:

```text
https://instagram.com/hmn
```

↓

```text
hmn
```

---

Example:

```text
https://x.com/hmn
```

↓

```text
hmn
```

---

Example:

```text
https://linkedin.com/in/hmn/
```

↓

```text
hmn
```

Only the identifier continues through the pipeline.

---

# Rule 4

## Remove Query Parameters

Tracking parameters are discarded.

Example:

```text
https://instagram.com/hmn?utm_source=campaign
```

↓

```text
hmn
```

---

# Rule 5

## Remove URL Fragments

Fragments are discarded.

Example:

```text
https://instagram.com/hmn#profile
```

↓

```text
hmn
```

---

# Rule 6

## Remove Trailing Slashes

Example:

```text
https://instagram.com/hmn/
```

↓

```text
hmn
```

---

# Rule 7

## Remove Platform Prefixes

Supported prefixes are removed when appropriate.

Example:

```text
@hmn
```

↓

```text
hmn
```

---

Example:

```text
/in/hmn
```

↓

```text
hmn
```

---

Only presentation prefixes are removed.

Identifier characters remain unchanged.

---

# Rule 8

## Apply Platform Case Rules

If a platform treats identifiers as case-insensitive, normalization stores the canonical lowercase form.

Example:

```text
HMN
```

↓

```text
hmn
```

If a future platform requires case preservation, its platform rule overrides this behavior.

---

# Rule 9

## Preserve Identifier Meaning

Normalization never inserts, removes, replaces, or guesses identifier characters.

Example:

```text
hmn
```

must never become

```text
hms
```

---

Example:

```text
john.dev
```

must never become

```text
john_dev
```

unless required by that platform's official identifier rules.

---

# Rule 10

## Reject Impossible Input

Normalization may reject input that cannot possibly represent a valid identifier.

Examples include:

* empty input
* unsupported URL format
* missing identifier
* invalid platform selection

Normalization rejects malformed input.

It never attempts to repair it.

---

# Platform Rules

Normalization does not contain platform-specific formatting logic.

Platform-specific behavior is defined exclusively inside:

```text
Social Platform Rules
```

Normalization simply executes those rules.

---

# Shared Engine

The normalization engine is shared by:

* Smart Mode
* Manual Mode

Both modes must always produce identical normalized identifiers when given the same input.

---

# Non Responsibilities

Normalization never performs:

* account discovery
* account existence checks
* ownership verification
* platform communication
* API requests
* web scraping
* browser automation
* AI correction
* spelling correction
* username guessing

---

# Design Principles

## Preserve Intent

Formatting may change.

Meaning never changes.

---

## Deterministic

The same input always produces the same normalized output.

---

## Platform Driven

Platform-specific rules live outside the normalization engine.

---

## Side Effect Free

Normalization never communicates with external services.

Its output depends only on:

* user input
* platform rules

---

## Shared Behavior

Every input source must use the same normalization engine.

There are no Smart Mode rules and Manual Mode rules.

There is only one normalization pipeline.

---

# Canonical Statement

Input Normalization is a deterministic formatting process that transforms user-provided social account identifiers into a standardized internal representation.

It removes presentation noise, extracts identifiers from supported URLs, applies platform formatting rules, and preserves user intent without performing discovery, verification, correction, or external communication.
