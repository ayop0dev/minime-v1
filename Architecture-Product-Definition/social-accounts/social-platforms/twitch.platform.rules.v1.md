# Twitch Platform Rules v1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the formatting rules for Twitch within the Social Accounts domain.

It specifies the identifier type, accepted input formats, display prefix, canonical public URL template, and normalization rules used to transform a user-provided identifier into a normalized Twitch social account record.

This document defines formatting only. It never defines account discovery, platform search, account existence checks, platform requests, API usage, scraping, browser automation, or verification. Removing internet access must not change this behavior.

These rules perform format validation only — they may reject a malformed identifier, an unsupported URL format, an unsupported prefix, or a missing identifier. They never perform account validation: they never determine whether an account exists, whether a username is available, or whether an account is valid, and they never verify anything against the platform. "Valid" and "invalid" below always refer to identifier format, never to account existence or status.

---

# Platform Identity

## Platform Name

Twitch

## Platform ID

twitch

## Rules Version

twitch.platform.rules.v1

---

# Identifier Type

Username

---

# Accepted Input

The following user inputs are accepted and normalized to the same identifier:

```text
hmn
https://twitch.tv/hmn
https://www.twitch.tv/hmn
```

---

# Display Prefix

None

---

# Canonical URL Template

```text
https://twitch.tv/{username}
```

Example:

```text
https://twitch.tv/ahmedofficial
```

---

# Identifier Rules

Allowed characters:

* a-z
* 0-9
* `_`

Formatting constraints:

* Minimum length: 4 characters
* Maximum length: 25 characters
* No spaces
* No periods
* No hyphens
* Stored in lowercase

Valid examples:

* ahmed
* ahmed_official
* ahmed2026

Invalid examples:

* ah
* ahmed.official
* ahmed-official

---

# Unsupported Input

The following are reserved Twitch paths and are not valid usernames:

```text
directory  downloads  jobs  settings  wallet
inventory  products  search  subs
```

---

# Normalization

* extract the username from supported channel URLs
* remove URL components
* lowercase the identifier

---

# Canonical Statement

Twitch Platform Rules define the canonical formatting behavior for Twitch social accounts: accepted input formats, normalization, display convention, and the canonical public URL template. They are completely independent from platform APIs, account discovery, verification, and external communication.
