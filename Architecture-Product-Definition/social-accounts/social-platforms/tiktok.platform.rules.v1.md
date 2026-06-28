# TikTok Platform Rules v1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the formatting rules for TikTok within the Social Accounts domain.

It specifies the identifier type, accepted input formats, display prefix, canonical public URL template, and normalization rules used to transform a user-provided identifier into a normalized TikTok social account record.

This document defines formatting only. It never defines account discovery, platform search, account existence checks, platform requests, API usage, scraping, browser automation, or verification. Removing internet access must not change this behavior.

These rules perform format validation only — they may reject a malformed identifier, an unsupported URL format, an unsupported prefix, or a missing identifier. They never perform account validation: they never determine whether an account exists, whether a username is available, or whether an account is valid, and they never verify anything against the platform. "Valid" and "invalid" below always refer to identifier format, never to account existence or status.

---

# Platform Identity

## Platform Name

TikTok

## Platform ID

tiktok

## Rules Version

tiktok.platform.rules.v1

---

# Identifier Type

Username

---

# Accepted Input

The following user inputs are accepted and normalized to the same identifier:

```text
hmn
@hmn
https://tiktok.com/@hmn
https://www.tiktok.com/@hmn
```

---

# Display Prefix

```text
@
```

---

# Canonical URL Template

```text
https://tiktok.com/@{username}
```

Example:

```text
https://tiktok.com/@ahmedofficial
```

---

# Identifier Rules

Allowed characters:

* a-z
* 0-9
* `.`
* `_`

Formatting constraints:

* Minimum length: 2 characters
* Maximum length: 24 characters
* No spaces
* No slashes
* Stored in lowercase

Valid examples:

* ahmedofficial
* ahmed.official
* ahmed_official

Invalid examples:

* ahmed official
* ahmed-official
* ahmed/official

---

# Unsupported Input

The following are reserved TikTok paths and are not valid usernames:

```text
login  signup  upload  discover  explore  following
live  inbox  messages  search  legal  privacy  terms
business  creators  ads
```

---

# Normalization

* remove leading `@`
* extract the username from supported profile URLs
* remove URL components
* lowercase the identifier

---

# Canonical Statement

TikTok Platform Rules define the canonical formatting behavior for TikTok social accounts: accepted input formats, normalization, display convention, and the canonical public URL template. They are completely independent from platform APIs, account discovery, verification, and external communication.
