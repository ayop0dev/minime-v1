# Authentication Policy V1

## Status

Approved

---

# Purpose

This document defines the authentication and verification rules used by Minime Account Claim System V1.

Authentication is responsible for verifying that a user controls the identity used during account creation.

Successful authentication converts a Username Reservation Record into an Approved User Account.

---

# Scope

This policy applies to:

* Account registration
* Account verification
* Initial account ownership
* Account login
* Identity verification

This policy does not define:

* Account security upgrades
* Two-factor authentication
* Recovery procedures
* Advanced account linking
* Risk analysis
* Fraud detection

Those systems may be introduced in future versions.

---

# Core Principles

## Provider-Based Authentication Only

Minime V1 authentication is provider-based only.

Minime never authenticates users directly.

Minime accepts successful authentication assertions from supported external identity providers only.

---

## Verification Before Account Creation

An account must never be created before successful authentication.

The sequence is:

```text
Username Reservation
↓
Provider Authentication
↓
Approved User Account
```

---

## One Verified Identity Required

Only one verified provider identity is required to create an account.

Either of the following is sufficient:

```text
Verified Google Identity
```

or

```text
Verified Apple Identity
```

---

# Supported Authentication Methods

## External Identity Providers

Minime V1 supports authentication exclusively through trusted external identity providers.

V1 supported providers:

```text
Google
Apple
```

Both providers are first-class and equal. Neither provider is preferred or ranked above the other.

---

## Google Sign-In

Users may register or sign in using Google Sign-In.

Successful Google authentication is considered verified immediately.

No additional verification step is required.

---

## Apple Sign-In

Users may register or sign in using Apple Sign-In.

Successful Apple authentication is considered verified immediately.

No additional verification step is required.

---

# Unsupported Authentication Methods

The following methods are not supported in V1:

```text
Email OTP
Phone Number Signup
Phone OTP
SMS Verification
WhatsApp Verification
Password Authentication
Facebook Login
X Login
LinkedIn Login
```

These methods are outside the scope of V1.

---

# Authentication Identity

Each account must have at least one authentication identity.

Authentication identities are external provider identities, not email addresses or passwords.

## Identity Fields

Each authentication identity record contains:

```text
provider          — the identity provider: "google" or "apple"
provider_subject  — the stable unique identifier issued by the provider (Google sub or Apple sub)
provider_email    — the email address reported by the provider; informational only; not the uniqueness key
```

## Uniqueness Rule

Authentication identities are unique by `(provider, provider_subject)`.

`provider_email` is informational only. It may change over the lifetime of a provider identity. It is never used as the uniqueness key.

---

# Account Uniqueness

Authentication identities may not be duplicated across accounts.

```text
Google Account A (sub: google_sub_123)
```

cannot create multiple Minime accounts.

```text
Apple Account A (sub: apple_sub_456)
```

cannot create multiple Minime accounts.

---

# Existing Account Detection

If an existing authentication identity attempts registration:

```text
Google Account Already Registered
```

or

```text
Apple Account Already Registered
```

the system must not create a second account.

The user should be directed to sign in to the existing account.

---

# Authentication Result

Authentication may return:

## success

Authentication completed successfully.

---

## failed

Authentication failed.

---

## expired

Authentication token expired.

---

## blocked

Authentication request blocked by system protections.

The `blocked` result is distinct from `failed`. A `failed` result indicates an incorrect credential. A `blocked` result indicates that the authentication request cannot proceed regardless of credential correctness.

The `blocked` state is triggered when the number of authentication attempts for a given identity exceeds the permitted threshold within a defined evaluation window. The Authentication Policy defines that a threshold and evaluation window must exist; their specific values are supplied by implementation configuration.

A blocked authentication context expires automatically. Re-authentication may be attempted only after the blocking period expires.

The Authentication Policy defines that the blocking period must not be indefinite and that every blocked state must have a defined expiry. The specific expiry duration is supplied by implementation configuration.

The authentication system owns enforcement of the blocked state. No Product Domain may override or bypass a blocked authentication result.

---

# Reservation Relationship

Authentication operates on an existing Username Reservation Record.

Authentication never creates usernames.

Authentication never selects usernames.

Authentication only verifies identity ownership.

---

# Successful Registration Flow

## Google Flow

```text
Username Available
↓
Create Reservation
↓
Google Sign-In
↓
Create Approved User Account
↓
Create Public Profile
```

---

## Apple Flow

```text
Username Available
↓
Create Reservation
↓
Apple Sign-In
↓
Create Approved User Account
↓
Create Public Profile
```

---

# Failed Registration Flow

```text
Username Available
↓
Create Reservation
↓
Authentication Not Completed
↓
Reservation Expires
↓
Username Returns To Available
```

---

# Future Versions

Future versions may introduce:

* Phone verification
* SMS OTP
* WhatsApp verification
* Multi-factor authentication
* Identity linking
* Account recovery
* Security levels
* Device trust systems
* Additional identity providers

These features are outside the scope of V1.
