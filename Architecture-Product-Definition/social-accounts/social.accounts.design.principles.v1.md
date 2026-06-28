# Social Accounts Design Principles V1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the architectural principles that govern every design decision within the Social Accounts domain.

Unlike implementation specifications, these principles describe **why** the domain is designed the way it is.

Every future feature, refactor, or architectural decision must remain consistent with these principles.

If a future proposal violates one or more of these principles, the proposal should be reconsidered before implementation.

---

# Philosophy

The Social Accounts domain exists to make social account collection simple, predictable, and maintainable.

It intentionally favors architectural simplicity over automation complexity.

The domain optimizes for:

* long-term stability
* deterministic behavior
* low maintenance
* zero platform dependency
* fast user onboarding

---

# Principle 1

## Trust User Intent

The user is the source of truth.

If a user says:

```text
My Instagram username is hmn
```

the system accepts that statement.

The system never attempts to determine whether the user is correct.

---

# Principle 2

## Normalize, Don't Correct

Normalization improves formatting.

Normalization never changes meaning.

Example:

```text
HMN
```

↓

```text
hmn
```

Allowed.

---

Example:

```text
hmn
```

↓

```text
hms
```

Forbidden.

The system never guesses user intent.

---

# Principle 3

## Generate, Don't Discover

The responsibility of the domain is to generate public profile URLs.

It is never responsible for discovering accounts.

The domain performs:

```text
Identifier

↓

Canonical URL
```

Nothing more.

---

# Principle 4

## Collection Before Verification

Collecting social accounts and verifying ownership are different responsibilities.

Users must always be able to create and publish their profile without proving ownership of any external account.

Verification, if introduced, is always optional and belongs to a separate domain.

---

# Principle 5

## Zero External Dependencies

The Social Accounts domain must never depend on:

* platform APIs
* web scraping
* browser automation
* search engines
* internet connectivity
* third-party services

The domain must remain fully operational even when every supported social platform is unavailable.

---

# Principle 6

## Platform Rules Own Platform Knowledge

Platform-specific knowledge belongs exclusively to the Platform Rules specification.

Other components must never hardcode:

* URL templates
* prefixes
* formatting rules
* platform conventions

Platform behavior should always be centralized.

---

# Principle 7

## Pure Components

Every component should perform one responsibility only.

Examples:

Normalization

```text
Input

↓

Normalized Identifier
```

---

URL Generation

```text
Identifier

↓

Canonical URL
```

---

Storage

```text
Canonical Record

↓

Database
```

No component should perform multiple unrelated responsibilities.

---

# Principle 8

## Deterministic Behavior

The same input must always produce the same output.

No result may depend on:

* timing
* location
* platform availability
* user session
* cookies
* internet connectivity

Behavior must be completely deterministic.

---

# Principle 9

## Canonical Data Only

The Social Accounts domain stores only canonical data owned by Minime.

Examples include:

* normalized identifiers
* canonical URLs
* visibility
* display order

External platform metadata is intentionally excluded.

---

# Principle 10

## Stateless Processing

Normalization and URL Generation are stateless transformations.

They remember nothing.

They cache nothing.

They depend only on:

* input
* platform rules

This guarantees predictability and simplifies testing.

---

# Principle 11

## Shared Logic

Smart Mode and Manual Mode are different user experiences.

They are not different processing engines.

Both modes must always use:

* the same normalization engine
* the same platform rules
* the same URL generation engine
* the same storage model

Only the method of collecting input differs.

---

# Principle 12

## Architecture Before Convenience

Short-term convenience must never introduce long-term architectural complexity.

Features that require:

* platform communication
* continuous maintenance
* scraping
* unstable integrations
* fragile automation

must be evaluated against the long-term simplicity of the domain before being accepted.

---

# Principle 13

## Stable by Design

The Social Accounts domain should require minimal maintenance over time.

Changes to external social platforms should rarely require modifications inside this domain.

The architecture should remain stable because it owns formatting—not platform behavior.

---

# Principle 14

## Explicit Responsibilities

Every responsibility belongs to exactly one component.

Examples:

Normalization

→ formatting

URL Generation

→ canonical URLs

Storage

→ persistence

OAuth

→ authorization

Rendering

→ presentation

Analytics

→ measurement

No responsibility should exist in multiple places.

---

# Principle 15

## Simplicity Is a Feature

The simplest architecture that satisfies the product requirements is the preferred architecture.

Additional automation is valuable only when it provides meaningful user benefit without increasing architectural complexity, maintenance cost, or platform dependency.

Complexity is never introduced simply because it is technically possible.

---

# Canonical Statement

The Social Accounts domain is intentionally designed as a deterministic, platform-independent, and low-maintenance architecture.

It trusts user intent, standardizes user input, generates canonical public profile URLs, and delivers Normalized Social Account Handoff Records to Connected Accounts for canonical storage.

Every architectural decision within this domain must preserve these principles before introducing additional functionality.
