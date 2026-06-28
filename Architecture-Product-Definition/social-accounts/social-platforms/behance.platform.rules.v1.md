# Behance Platform Rules v1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the formatting rules for Behance within the Social Accounts domain.

It specifies the identifier type, accepted input formats, display prefix, canonical public URL template, and normalization rules used to transform a user-provided identifier into a normalized Behance social account record.

This document defines formatting only. It never defines account discovery, platform search, account existence checks, platform requests, API usage, scraping, browser automation, or verification. Removing internet access must not change this behavior.

These rules perform format validation only — they may reject a malformed identifier, an unsupported URL format, an unsupported prefix, or a missing identifier. They never perform account validation: they never determine whether an account exists, whether a username is available, or whether an account is valid, and they never verify anything against the platform. "Valid" and "invalid" below always refer to identifier format, never to account existence or status.

---

# Platform Identity

## Platform Name

Behance

## Platform ID

behance

## Rules Version

behance.platform.rules.v1

---

# Identifier Type

Username

---

# Accepted Input

The following user inputs are accepted and normalized to the same identifier:

```text
hmn
https://behance.net/hmn
https://www.behance.net/hmn
```

---

# Display Prefix

None

---

# Canonical URL Template

```text
https://behance.net/{username}
```

Example:

```text
https://behance.net/ahmedofficial
```

---

# Identifier Rules

Allowed characters:

* a-z
* 0-9

Formatting constraints:

* No spaces
* No special characters
* Stored in lowercase

Valid examples:

* ahmed
* ahmedofficial
* ahmed2026

Invalid examples:

* ahmed official
* ahmed@
* ahmed#

---

# Unsupported Input

The following are reserved Behance paths and are not valid usernames:

```text
gallery  search  joblist  assets  hire  career  about
```

The following URL forms are not profile identifiers and are not supported:

```text
behance.net/gallery/
behance.net/joblist/
behance.net/search/
behance.net/challenges/
```

---

# Normalization

* extract the username from supported profile URLs
* remove URL components
* lowercase the identifier

---

# Canonical Statement

Behance Platform Rules define the canonical formatting behavior for Behance social accounts: accepted input formats, normalization, display convention, and the canonical public URL template. They are completely independent from platform APIs, account discovery, verification, and external communication.
