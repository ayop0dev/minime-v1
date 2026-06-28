# Telegram Platform Rules v1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the formatting rules for Telegram within the Social Accounts domain.

It specifies the identifier type, accepted input formats, display prefix, canonical public URL template, and normalization rules used to transform a user-provided identifier into a normalized Telegram social account record.

This document defines formatting only. It never defines account discovery, platform search, account existence checks, platform requests, API usage, scraping, browser automation, or verification. Removing internet access must not change this behavior.

These rules perform format validation only — they may reject a malformed identifier, an unsupported URL format, an unsupported prefix, or a missing identifier. They never perform account validation: they never determine whether an account exists, whether a username is available, or whether an account is valid, and they never verify anything against the platform. "Valid" and "invalid" below always refer to identifier format, never to account existence or status.

---

# Platform Identity

## Platform Name

Telegram

## Platform ID

telegram

## Rules Version

telegram.platform.rules.v1

---

# Identifier Type

Username

---

# Accepted Input

The following user inputs are accepted and normalized to the same identifier:

```text
hmn
@hmn
https://t.me/hmn
```

---

# Display Prefix

```text
@
```

---

# Canonical URL Template

```text
https://t.me/{username}
```

Example:

```text
https://t.me/ahmedofficial
```

---

# Identifier Rules

Allowed characters:

* a-z
* 0-9
* `_`

Formatting constraints:

* Minimum length: 5 characters
* Maximum length: 32 characters
* Must start with a letter
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
* 12345
* ahmed.official
* ahmed-official

---

# Unsupported Input

The following are reserved Telegram paths and are not valid usernames:

```text
login  share  proxy  faq  privacy  blog  apps  tour  contact
```

The following URL forms are not username identifiers and are not supported:

```text
t.me/joinchat/
t.me/+invitecode
telegram.me/addstickers/
telegram.me/share/
telegram.me/proxy/
```

---

# Normalization

* remove leading `@`
* extract the username from supported `t.me` URLs
* remove URL components
* lowercase the identifier

---

# Canonical Statement

Telegram Platform Rules define the canonical formatting behavior for Telegram social accounts: accepted input formats, normalization, display convention, and the canonical public URL template. They are completely independent from platform APIs, account discovery, verification, and external communication.
