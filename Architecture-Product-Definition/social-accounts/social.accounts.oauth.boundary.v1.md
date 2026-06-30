# Social Accounts OAuth Boundary Specification V1

**Status:** Approved
**Version:** V1
**Domain:** Social Accounts

---

# Purpose

This document defines the architectural boundary between the Social Accounts domain and future OAuth-based platform integrations.

OAuth is intentionally excluded from the Social Accounts domain.

The purpose of this document is to ensure that future platform integrations do not introduce responsibility overlap or increase the complexity of social account collection.

---

# Design Philosophy

Social account collection and platform authorization are different responsibilities.

Collecting a social account identifier is not the same as proving ownership of that account.

These concerns must remain independent.

---

# Social Accounts Responsibility

The Social Accounts domain is responsible for:

* collecting identifiers
* normalizing identifiers
* generating canonical public profile URLs
* storing social account records

Its responsibility ends immediately after the social account record has been stored.

---

# OAuth Responsibility

OAuth is responsible for official platform authorization.

Future OAuth integrations may support:

* ownership verification
* platform synchronization
* optional profile import
* advanced platform integrations
* future AI features

These capabilities are intentionally outside the Social Accounts domain.

---

# User Experience

OAuth is never required during onboarding.

Users must always be able to create and publish their public profile without connecting any external social platform.

OAuth is an optional enhancement.

---

# Settings Integration

If OAuth is introduced in future versions, it belongs inside:

```text
Settings

↓

Connected Accounts
```

It is never part of:

```text
Social Accounts Setup
```

---

# Relationship With Existing Social Accounts

OAuth never creates social account records automatically.

Existing social accounts remain user-owned records.

OAuth may later become associated with an existing record, but it never replaces user-provided identifiers.

---

# Data Ownership

Social Accounts stores:

* identifiers
* canonical URLs
* presentation metadata

OAuth stores:

* authorization state
* provider identifiers
* access tokens
* refresh tokens
* synchronization metadata

Neither domain stores the other's data.

---

# Independent Lifecycles

Deleting a social account does not revoke OAuth authorization.

Disconnecting OAuth does not remove the user's social account unless the user explicitly chooses to do so.

The two systems remain independent.

---

# Future Compatibility

Future features such as:

* profile synchronization
* analytics enrichment
* AI profile reading
* platform import

must consume existing social account records instead of replacing the Social Accounts domain.

---

# Design Principles

## Optional

OAuth is never mandatory.

---

## Independent

OAuth and Social Accounts have separate responsibilities.

---

## Replace Nothing

OAuth enhances existing records.

It never replaces them.

---

## No Hidden Side Effects

Connecting a platform must never silently:

* create accounts
* modify identifiers
* change URLs
* reorder accounts
* change visibility

Any future synchronization must always be explicit.

---

## Domain Isolation

The Social Accounts domain must remain fully functional even if OAuth integrations are disabled or unavailable.

---

# Canonical Statement

OAuth is an optional platform authorization capability that exists outside the Social Accounts domain.

The Social Accounts domain remains solely responsible for collecting, normalizing, generating, and storing user-provided social account identifiers, while OAuth independently manages platform authorization and future integration features.
