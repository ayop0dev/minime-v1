# YouTube Platform Rules v1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the formatting rules for YouTube within the Social Accounts domain.

It specifies the identifier type, accepted input formats, display prefix, canonical public URL template, and normalization rules used to transform a user-provided identifier into a normalized YouTube social account record.

This document defines formatting only. It never defines account discovery, platform search, account existence checks, platform requests, API usage, scraping, browser automation, or verification. Removing internet access must not change this behavior.

These rules perform format validation only — they may reject a malformed identifier, an unsupported URL format, an unsupported prefix, or a missing identifier. They never perform account validation: they never determine whether an account exists, whether a username is available, or whether an account is valid, and they never verify anything against the platform. "Valid" and "invalid" below always refer to identifier format, never to account existence or status.

---

# Platform Identity

## Platform Name

YouTube

## Platform ID

youtube

## Rules Version

youtube.platform.rules.v1

---

# Identifier Type

Handle

YouTube Platform Rules cover handle channels only (`@handle`). Legacy `/user/`, `/c/`, and `/channel/` URL forms are out of scope.

---

# Accepted Input

The following user inputs are accepted and normalized to the same identifier:

```text
hmn
@hmn
https://youtube.com/@hmn
https://www.youtube.com/@hmn
```

---

# Display Prefix

```text
@
```

---

# Canonical URL Template

```text
https://youtube.com/@{handle}
```

Example:

```text
https://youtube.com/@ahmedofficial
```

---

# Identifier Rules

Allowed characters:

* a-z
* 0-9
* `.`
* `_`
* `-`

Formatting constraints:

* Minimum length: 3 characters
* Maximum length: 30 characters
* No spaces
* No slashes
* Stored in lowercase

Valid examples:

* ahmedofficial
* ahmed.official
* ahmed_official
* ahmed-official

Invalid examples:

* ahmed official
* ahmed/official

---

# Unsupported Input

The following are reserved YouTube paths and are not valid handles:

```text
watch  shorts  feed  results  playlist  channel
user  c  premium  gaming  music  kids  live  upload
account  studio  create  about
```

The following URL forms are not handle identifiers and are not supported:

```text
youtube.com/user/
youtube.com/c/
youtube.com/channel/
```

---

# Normalization

* remove leading `@`
* extract the handle from supported channel URLs
* remove URL components
* lowercase the identifier

---

# Canonical Statement

YouTube Platform Rules define the canonical formatting behavior for YouTube social accounts: accepted input formats, normalization, display convention, and the canonical public URL template. They are completely independent from platform APIs, account discovery, verification, and external communication.
