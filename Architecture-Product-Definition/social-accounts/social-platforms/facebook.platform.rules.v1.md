# Facebook Platform Rules v1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the formatting rules for Facebook within the Social Accounts domain.

It specifies the identifier type, accepted input formats, display prefix, canonical public URL template, and normalization rules used to transform a user-provided identifier into a normalized Facebook social account record.

This document defines formatting only. It never defines account discovery, platform search, account existence checks, platform requests, API usage, scraping, browser automation, or verification. Removing internet access must not change this behavior.

These rules perform format validation only — they may reject a malformed identifier, an unsupported URL format, an unsupported prefix, or a missing identifier. They never perform account validation: they never determine whether an account exists, whether a username is available, or whether an account is valid, and they never verify anything against the platform. "Valid" and "invalid" below always refer to identifier format, never to account existence or status.

---

# Platform Identity

## Platform Name

Facebook

## Platform ID

facebook

## Rules Version

facebook.platform.rules.v1

---

# Identifier Type

Username

---

# Accepted Input

The following user inputs are accepted and normalized to the same identifier:

```text
hmn
https://facebook.com/hmn
https://www.facebook.com/hmn
```

---

# Display Prefix

None

---

# Canonical URL Template

```text
https://facebook.com/{username}
```

Example:

```text
https://facebook.com/ahmedofficial
```

---

# Identifier Rules

Allowed characters:

* a-z
* 0-9
* `.`

Formatting constraints:

* Minimum length: 5 characters
* No spaces
* No underscores
* No hyphens
* No slashes
* Stored in lowercase

Valid examples:

* ahmedofficial
* ahmed.official
* ahmed.salem.25

Invalid examples:

* ahme
* ahmed official
* ahmed_official
* ahmed-official

Note: Facebook treats periods as non-distinguishing characters. This is a display nuance only and is never used to merge or relate accounts across platforms.

---

# Unsupported Input

The following are reserved Facebook paths and are not valid usernames:

```text
home  profile  pages  groups  events  marketplace
watch  gaming  reels  stories  friends  messages
settings  privacy  help  business  ads  login  signup
```

The following URL forms are not personal-profile identifiers and are not supported:

```text
facebook.com/pages/
facebook.com/groups/
facebook.com/events/
facebook.com/profile.php?id=
```

---

# Normalization

* extract the username from supported profile URLs
* remove URL components
* lowercase the identifier

---

# Canonical Statement

Facebook Platform Rules define the canonical formatting behavior for Facebook social accounts: accepted input formats, normalization, display convention, and the canonical public URL template. They are completely independent from platform APIs, account discovery, verification, and external communication.
