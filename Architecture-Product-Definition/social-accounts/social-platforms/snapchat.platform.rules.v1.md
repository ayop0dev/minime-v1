# Snapchat Platform Rules v1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the formatting rules for Snapchat within the Social Accounts domain.

It specifies the identifier type, accepted input formats, display prefix, canonical public URL template, and normalization rules used to transform a user-provided identifier into a normalized Snapchat social account record.

This document defines formatting only. It never defines account discovery, platform search, account existence checks, platform requests, API usage, scraping, browser automation, or verification. Removing internet access must not change this behavior.

These rules perform format validation only — they may reject a malformed identifier, an unsupported URL format, an unsupported prefix, or a missing identifier. They never perform account validation: they never determine whether an account exists, whether a username is available, or whether an account is valid, and they never verify anything against the platform. "Valid" and "invalid" below always refer to identifier format, never to account existence or status.

---

# Platform Identity

## Platform Name

Snapchat

## Platform ID

snapchat

## Rules Version

snapchat.platform.rules.v1

---

# Identifier Type

Username

---

# Accepted Input

The following user inputs are accepted and normalized to the same identifier:

```text
hmn
https://snapchat.com/add/hmn
https://www.snapchat.com/add/hmn
```

---

# Display Prefix

None

---

# Canonical URL Template

```text
https://snapchat.com/add/{username}
```

Example:

```text
https://snapchat.com/add/ahmedofficial
```

---

# Identifier Rules

Allowed characters:

* a-z
* 0-9
* `_`
* `.`

Formatting constraints:

* Minimum length: 3 characters
* Maximum length: 15 characters
* No spaces
* No special characters
* Stored in lowercase

Valid examples:

* ahmed
* ahmed_official
* ahmed.official

Invalid examples:

* ah med
* ahmed@
* ahmed#

---

# Unsupported Input

The following are reserved Snapchat paths and are not valid usernames:

```text
discover  spotlight  lens  stories  ads  business  creators
```

The following URL forms are not profile identifiers and are not supported:

```text
snapchat.com/t/
snapchat.com/unlock/
snapchat.com/spotlight/
snapchat.com/discover/
snapchat.com/lens/
```

---

# Normalization

* extract the username from supported `add/` profile URLs
* remove URL components
* lowercase the identifier

---

# Canonical Statement

Snapchat Platform Rules define the canonical formatting behavior for Snapchat social accounts: accepted input formats, normalization, display convention, and the canonical public URL template. They are completely independent from platform APIs, account discovery, verification, and external communication.
