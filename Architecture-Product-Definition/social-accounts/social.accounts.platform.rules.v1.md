# Social Platform Rules Specification V1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the platform-specific formatting rules used by the Social Accounts domain.

Each supported platform provides its own identifier format, URL template, input guidance, and normalization behavior.

The purpose of this specification is to centralize all platform-specific rules while keeping the normalization engine completely platform-independent.

---

# Authoritative Platform Registry

The authoritative, per-platform rule set lives in the `social-accounts/platform/` directory as one `<platform>.platform.rules.v1.md` file per platform.

This document defines the **shared processing model** and the canonical formatting fields (identifier type, accepted input, display prefix, URL template, normalization). The platform sections below are kept consistent with the per-platform rule files.

Precedence rule: where this document and a per-platform rule file differ for the same platform, **the per-platform rule file is authoritative**. New platform support is added by creating a per-platform rule file with the six required fields; it is never added by inference, lookup, or any external communication.

---

# Design Philosophy

Platform Rules define formatting.

They never define behavior.

This document specifies:

* identifier format
* input examples
* display guidance
* URL generation
* normalization behavior

This document never specifies:

* API integration
* platform communication
* account verification
* account discovery
* scraping
* browser automation

---

# Validation Scope

Platform Rules perform **Format Validation only**. They never perform **Account Validation**.

Format Validation (allowed) may reject:

* a malformed identifier
* an unsupported URL format
* an unsupported prefix
* a missing identifier

Account Validation (forbidden) is never performed. Platform Rules never determine:

* whether an account exists
* whether an account does not exist
* whether a username is available
* whether a username is unavailable
* whether an account is valid or invalid
* anything that requires verification against the platform

"Valid" and "invalid" in this document always refer to the **format of the identifier**, never to the existence or status of an account.

---

# Shared Rule

Every platform follows the same processing model.

```text
User Input

↓

Normalize

↓

Platform Rules

↓

Public URL

↓

Save
```

Platform Rules never communicate with external services.

---

# Instagram

## Platform Name

Instagram

---

## Identifier Type

Username

---

## User Input

Accepted:

```text
hmn

@hmn

https://instagram.com/hmn

https://www.instagram.com/hmn/
```

---

## Display Prefix

```text
@
```

---

## URL Template

```text
https://instagram.com/{username}
```

---

## Normalization

* remove leading @
* remove URL components
* remove trailing slash
* remove query parameters
* lowercase identifier

---

# TikTok

## Platform Name

TikTok

---

## Identifier Type

Username

---

## User Input

Accepted:

```text
hmn

@hmn

https://tiktok.com/@hmn
```

---

## Display Prefix

```text
@
```

---

## URL Template

```text
https://tiktok.com/@{username}
```

---

## Normalization

* remove leading @
* remove URL components
* lowercase identifier

---

# X

## Platform Name

X

---

## Identifier Type

Username

---

## User Input

Accepted:

```text
hmn

@hmn

https://x.com/hmn
```

---

## Display Prefix

```text
@
```

---

## URL Template

```text
https://x.com/{username}
```

---

## Normalization

* remove leading @
* remove URL components
* lowercase identifier

---

# Facebook

## Platform Name

Facebook

---

## Identifier Type

Username

---

## User Input

Accepted:

```text
hmn

https://facebook.com/hmn
```

---

## Display Prefix

None

---

## URL Template

```text
https://facebook.com/{username}
```

---

## Normalization

* remove URL components
* lowercase identifier

---

# LinkedIn

## Platform Name

LinkedIn

---

## Identifier Type

Public Profile Identifier

---

## User Input

Accepted:

```text
hmn

/in/hmn

https://linkedin.com/in/hmn
```

---

## Display Prefix

```text
/in/
```

---

## URL Template

```text
https://linkedin.com/in/{username}
```

---

## Normalization

* remove /in/
* remove URL components
* lowercase identifier

---

# YouTube

## Platform Name

YouTube

---

## Identifier Type

Handle

---

## User Input

Accepted:

```text
hmn

@hmn

https://youtube.com/@hmn
```

---

## Display Prefix

```text
@
```

---

## URL Template

```text
https://youtube.com/@{handle}
```

---

## Normalization

* remove leading @
* remove URL components
* lowercase identifier

---

# GitHub

## Platform Name

GitHub

---

## Identifier Type

Username

---

## User Input

Accepted:

```text
hmn

https://github.com/hmn
```

---

## Display Prefix

None

---

## URL Template

```text
https://github.com/{username}
```

---

## Normalization

* remove URL components
* lowercase identifier

---

# Pinterest

## Platform Name

Pinterest

---

## Identifier Type

Username

---

## User Input

Accepted:

```text
hmn

https://pinterest.com/hmn
```

---

## Display Prefix

None

---

## URL Template

```text
https://pinterest.com/{username}
```

---

## Normalization

* remove URL components
* lowercase identifier

---

# Behance

## Platform Name

Behance

---

## Identifier Type

Username

---

## User Input

Accepted:

```text
hmn

https://behance.net/hmn
```

---

## Display Prefix

None

---

## URL Template

```text
https://behance.net/{username}
```

---

## Normalization

* remove URL components
* lowercase identifier

---

# Future Platforms

Adding a new platform must require only:

1. Platform Name
2. Identifier Type
3. Accepted Input Formats
4. Display Prefix
5. URL Template
6. Normalization Rules

No modification should be required inside:

* Smart Mode
* Manual Mode
* Input Normalization
* Storage
* Rendering

The platform registry should be extensible without changing the surrounding architecture.

---

# Design Principles

## Formatting Only

Platform Rules define formatting.

Never behavior.

---

## Stateless

Platform Rules depend only on:

* user input
* platform definition

They never depend on:

* internet connectivity
* APIs
* platform availability

---

## Canonical URLs

Every generated URL must use Minime's canonical public URL template for the platform.

---

## Platform Isolation

Each platform definition is completely independent.

Updating one platform must never affect another.

---

## Extensible

Supporting additional social platforms must only require registering a new platform definition.

No architectural redesign should be necessary.

---

# Canonical Statement

Social Platform Rules define the canonical formatting behavior for every supported social platform.

They specify accepted input formats, normalization behavior, display conventions, and public URL templates while remaining completely independent from platform APIs, account discovery, verification, or external communication.
