# LinkedIn Platform Rules v1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the formatting rules for LinkedIn within the Social Accounts domain.

It specifies the identifier type, accepted input formats, display prefix, canonical public URL template, and normalization rules used to transform a user-provided identifier into a normalized LinkedIn social account record.

This document defines formatting only. It never defines account discovery, platform search, account existence checks, platform requests, API usage, scraping, browser automation, or verification. Removing internet access must not change this behavior.

These rules perform format validation only — they may reject a malformed identifier, an unsupported URL format, an unsupported prefix, or a missing identifier. They never perform account validation: they never determine whether an account exists, whether a username is available, or whether an account is valid, and they never verify anything against the platform. "Valid" and "invalid" below always refer to identifier format, never to account existence or status.

---

# Platform Identity

## Platform Name

LinkedIn

## Platform ID

linkedin

## Rules Version

linkedin.platform.rules.v1

---

# Identifier Type

Public Profile Identifier

LinkedIn Platform Rules cover personal public profile identifiers only (`/in/` profiles). Company, school, and showcase pages are out of scope.

---

# Accepted Input

The following user inputs are accepted and normalized to the same identifier:

```text
hmn
/in/hmn
https://linkedin.com/in/hmn
https://www.linkedin.com/in/hmn/
```

---

# Display Prefix

```text
/in/
```

---

# Canonical URL Template

```text
https://linkedin.com/in/{username}
```

Example:

```text
https://linkedin.com/in/ahmed-salem
```

---

# Identifier Rules

Allowed characters:

* a-z
* 0-9
* `-`

Formatting constraints:

* Minimum length: 3 characters
* Maximum length: 100 characters
* No spaces
* No underscores
* No slashes
* Stored in lowercase

Valid examples:

* ahmed-salem
* ahmed-salem-25
* mohamed-hassan

Invalid examples:

* ahmed salem
* ahmed_salem
* ahmed/salem

---

# Unsupported Input

The following are reserved LinkedIn paths and are not valid profile identifiers:

```text
feed  jobs  learning  groups  events  posts  pulse
help  about  legal  privacy  settings  messaging
company  school  showcase  signin  signup
```

The following URL forms are not personal-profile identifiers and are not supported:

```text
linkedin.com/company/
linkedin.com/school/
linkedin.com/showcase/
```

---

# Normalization

* remove leading `/in/`
* extract the identifier from supported profile URLs
* remove URL components
* remove trailing slash
* lowercase the identifier

---

# Canonical Statement

LinkedIn Platform Rules define the canonical formatting behavior for LinkedIn social accounts: accepted input formats, normalization, display convention, and the canonical public URL template. They are completely independent from platform APIs, account discovery, verification, and external communication.
