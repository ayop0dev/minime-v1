# Spotify Platform Rules v1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the formatting rules for Spotify within the Social Accounts domain.

It specifies the identifier type, accepted input formats, display prefix, canonical public URL template, and normalization rules used to transform a user-provided identifier into a normalized Spotify social account record.

This document defines formatting only. It never defines account discovery, platform search, account existence checks, platform requests, API usage, scraping, browser automation, or verification. Removing internet access must not change this behavior.

These rules perform format validation only — they may reject a malformed identifier, an unsupported URL format, an unsupported prefix, or a missing identifier. They never perform account validation: they never determine whether an account exists, whether a username is available, or whether an account is valid, and they never verify anything against the platform. "Valid" and "invalid" below always refer to identifier format, never to account existence or status.

---

# Platform Identity

## Platform Name

Spotify

## Platform ID

spotify

## Rules Version

spotify.platform.rules.v1

---

# Identifier Type

User Profile Identifier

Spotify user identifiers are platform-assigned. Spotify does not publish deterministic username formatting rules, so the identifier is accepted as provided after generic normalization. No format-based validation is applied.

---

# Accepted Input

The following user inputs are accepted and normalized to the same identifier:

```text
hmn
https://open.spotify.com/user/hmn
```

---

# Display Prefix

None

---

# Canonical URL Template

```text
https://open.spotify.com/user/{username}
```

Example:

```text
https://open.spotify.com/user/ahmedofficial
```

---

# Identifier Rules

* Spotify user identifiers are platform-assigned.
* No format-based validation is applied.
* Stored in lowercase for consistency.

---

# Unsupported Input

The following URL forms are not user-profile identifiers and are not supported:

```text
open.spotify.com/artist/
open.spotify.com/album/
open.spotify.com/track/
open.spotify.com/show/
open.spotify.com/episode/
open.spotify.com/playlist/
```

---

# Normalization

* extract the user identifier from supported `user/` URLs
* remove URL components
* lowercase the identifier

---

# Canonical Statement

Spotify Platform Rules define the canonical formatting behavior for Spotify social accounts: accepted input formats, normalization, display convention, and the canonical public URL template. They are completely independent from platform APIs, account discovery, verification, and external communication.
