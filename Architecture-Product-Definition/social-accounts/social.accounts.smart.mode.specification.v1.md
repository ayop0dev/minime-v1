# Social Accounts Smart Mode Specification V1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

Smart Mode exists to reduce user effort during social account setup.

It allows users who use the same (or nearly the same) username across multiple platforms to create multiple social account links from a single input.

Smart Mode is an input automation feature.

It is **not** a discovery feature.

---

# Design Goal

The primary objective is to eliminate repetitive typing.

Instead of asking the user to enter the same username multiple times, Smart Mode allows the user to provide it once and automatically generates the public profile URLs for the selected platforms.

---

# Core Philosophy

Smart Mode assumes:

> The user already knows their own social account identifier.

The system does not attempt to determine whether the identifier is correct.

It simply helps the user reuse it efficiently.

---

# Supported Inputs

Smart Mode accepts two input types.

## Username

Example:

```text
hmn
```

---

## Public Profile URL

Example:

```text
https://instagram.com/HMn/?utm_source=campaign
```

```text
https://x.com/HMn
```

```text
https://youtube.com/@HMn
```

The input may contain:

* uppercase characters
* trailing slashes
* tracking parameters
* platform prefixes
* full URLs

All supported formats are normalized before processing.

---

# Processing Pipeline

The Smart Mode pipeline is intentionally simple.

```text
User Input

↓

Normalize Input

↓

Extract Identifier

↓

Apply Platform Rules

↓

Generate Platform URLs

↓

Save Social Accounts
```

No additional processing is performed.

---

# Username Extraction

If the user provides a public profile URL, Smart Mode extracts only the platform identifier.

Example:

```text
Input

https://instagram.com/HMn/?utm_source=abc

↓

Identifier

HMn

↓

Normalized

hmn
```

Only the identifier continues through the pipeline.

Everything else is discarded.

---

# Input Normalization

Smart Mode normalizes identifiers before URL generation.

Typical normalization includes:

* trim whitespace
* remove tracking parameters
* remove unnecessary trailing slashes
* remove leading platform symbols when applicable
* apply case normalization according to platform rules

Normalization exists only to standardize formatting.

It never attempts to correct mistakes.

---

# URL Generation

After normalization, Smart Mode generates public URLs using each platform's official URL template.

Example:

```text
Identifier

hmn

↓

Instagram

https://instagram.com/hmn

↓

TikTok

https://tiktok.com/@hmn

↓

X

https://x.com/hmn

↓

YouTube

https://youtube.com/@hmn

↓

LinkedIn

https://linkedin.com/in/hmn
```

The URL templates themselves are defined separately by the Platform Rules specification.

---

# Platform Selection

Users may choose which platforms should receive the generated identifier.

Smart Mode only generates URLs for the selected platforms.

It never creates accounts automatically for every supported platform without user selection.

---

# Save Behavior

After URL generation:

* a Normalized Social Account Handoff Record is produced
* the handoff record is delivered to Connected Accounts
* Connected Accounts creates the canonical Connected Account record

Saving is immediate.

There is:

* no preview
* no draft
* no publish confirmation

---

# What Smart Mode Does

Smart Mode:

* accepts one identifier
* accepts one supported profile URL
* extracts identifiers
* normalizes formatting
* applies platform rules
* generates public URLs
* produces Normalized Social Account Handoff Records
* delivers handoff records to Connected Accounts

---

# What Smart Mode Never Does

Smart Mode never:

* searches platforms
* checks username availability
* verifies account existence
* validates ownership
* performs OAuth
* calls APIs
* opens public profile pages
* performs web scraping
* launches browser automation
* queries search engines
* ranks confidence
* guesses usernames
* modifies user intent

---

# User Responsibility

The user is responsible for providing the intended identifier.

Example:

```text
User enters

hmn
```

If the intended identifier was actually:

```text
hms
```

The system must preserve:

```text
hmn
```

The system never attempts to infer user intent.

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
https://instagram.com/HMN/?utm_source=test
```

↓

```text
hmn
```

Formatting may change.

Meaning must never change.

---

# Design Principles

## Reduce Typing

One input should produce multiple correctly formatted profile URLs.

---

## Trust User Intent

The identifier belongs to the user.

The system does not reinterpret it.

---

## Normalize, Don't Correct

Formatting may change.

Meaning never changes.

---

## Generate, Don't Discover

Smart Mode constructs URLs.

It never discovers accounts.

---

## Zero Platform Dependency

Smart Mode must continue functioning even if every supported social platform is temporarily unavailable.

Its behavior depends only on user input and platform formatting rules.

---

# Canonical Statement

Smart Mode is a username and URL automation mechanism.

It extracts, normalizes, and reuses a user-provided identifier to generate standardized public social profile URLs.

It never performs account discovery, platform communication, account verification, or account existence validation.
