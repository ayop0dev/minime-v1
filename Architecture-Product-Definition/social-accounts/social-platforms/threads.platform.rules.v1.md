# Threads Platform Rules v1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the formatting rules for Threads within the Social Accounts domain.

It specifies the identifier type, accepted input formats, display prefix, canonical public URL template, and normalization rules used to transform a user-provided identifier into a normalized Threads social account record.

This document defines formatting only. It never defines account discovery, platform search, account existence checks, platform requests, API usage, scraping, browser automation, or verification. Removing internet access must not change this behavior.

These rules perform format validation only — they may reject a malformed identifier, an unsupported URL format, an unsupported prefix, or a missing identifier. They never perform account validation: they never determine whether an account exists, whether a username is available, or whether an account is valid, and they never verify anything against the platform. "Valid" and "invalid" below always refer to identifier format, never to account existence or status.

---

# Platform Identity

## Platform Name

Threads

## Platform ID

threads

## Rules Version

threads.platform.rules.v1

---

# Identifier Type

Username

---

# Accepted Input

The following user inputs are accepted and normalized to the same identifier:

```text
hmn
@hmn
https://threads.net/@hmn
https://www.threads.net/@hmn
```

---

# Display Prefix

```text
@
```

---

# Canonical URL Template

```text
https://threads.net/@{username}
```

Example:

```text
https://threads.net/@ahmedofficial
```

---

# Identifier Rules

Threads identifiers follow Instagram username conventions.

Allowed characters:

* a-z
* 0-9
* `_`
* `.`

Formatting constraints:

* Minimum length: 1 character
* Maximum length: 30 characters
* No spaces
* No hyphens
* Stored in lowercase

Valid examples:

* ahmed
* ahmed_official
* ahmed.official

Invalid examples:

* ahmed official
* ahmed-official
* ahmed@

---

# Unsupported Input

The following are reserved Threads paths and are not valid usernames:

```text
about  search  login  signup  explore  privacy  terms  help
```

The following URL forms are not profile identifiers and are not supported:

```text
threads.net/t/
threads.net/search/
threads.net/topic/
threads.net/tag/
```

---

# Normalization

* remove leading `@`
* extract the username from supported profile URLs
* remove URL components
* lowercase the identifier

---

# Canonical Statement

Threads Platform Rules define the canonical formatting behavior for Threads social accounts: accepted input formats, normalization, display convention, and the canonical public URL template. They are completely independent from platform APIs, account discovery, verification, and external communication.
