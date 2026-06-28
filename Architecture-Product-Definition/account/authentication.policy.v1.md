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

## Zero Variable Cost

Authentication V1 must maintain:

```text
Variable Cost ≈ 0
```

Authentication must not depend on:

* SMS verification
* Paid verification providers
* Paid identity providers
* Third-party fraud services

---

## Verification Before Account Creation

An account must never be created before successful authentication.

The sequence is:

```text
Username Reservation
↓
Authentication
↓
Verification
↓
Approved User Account
```

---

## One Verified Identity Required

Only one verified identity is required to create an account.

Examples:

```text
Verified Email
```

or

```text
Verified Google Account
```

Both are sufficient.

---

# Supported Authentication Methods

## Email OTP

Users may register using an email address.

Authentication occurs through a one-time password delivered by email.

Successful OTP verification completes authentication.

---

## Google Sign-In

Users may register using Google Sign-In.

Successful Google authentication is considered verified immediately.

No additional OTP is required.

---

# Unsupported Authentication Methods

The following methods are not supported in V1:

```text
Phone Number Signup
Phone OTP
SMS Verification
WhatsApp Verification
Apple Sign-In
Facebook Login
X Login
LinkedIn Login
```

These methods are outside the scope of V1.

---

# Email OTP

## Purpose

Email OTP verifies ownership of an email address.

---

## OTP Format

OTP must be:

```text
6 digits
numeric only
```

Example:

```text
481927
```

---

## OTP Lifetime

OTP validity:

```text
10 minutes
```

After expiration:

```text
expired
```

The user must request a new OTP.

---

## Maximum Attempts

Maximum verification attempts:

```text
5
```

After reaching the limit:

```text
OTP invalidated
```

A new OTP must be generated.

---

## Resend Cooldown

Minimum resend delay:

```text
60 seconds
```

between OTP requests.

---

## OTP Storage

Plain-text OTP values must never be stored.

Only hashed verification values should be stored.

---

# Google Authentication

## Verification

Successful Google authentication is considered verified immediately.

No secondary verification is required.

---

## Account Creation

After successful Google authentication:

```text
Username Reservation
↓
Google Authentication
↓
Approved User Account
```

---

# Approved User Account Creation

An account becomes approved when one supported authentication method succeeds.

Examples:

```text
Username Reservation
↓
Email OTP Verified
↓
Approved User Account
```

or

```text
Username Reservation
↓
Google Authentication
↓
Approved User Account
```

---

# Authentication Identity

Each account must have a primary authentication identity.

Examples:

```text
Email Identity
```

or

```text
Google Identity
```

---

# Account Uniqueness

Authentication identities may not be duplicated across accounts.

Examples:

```text
user@example.com
```

cannot create multiple Minime accounts.

---

```text
Google Account A
```

cannot create multiple Minime accounts.

---

# Existing Account Detection

If an existing authentication identity attempts registration:

```text
Email Already Registered
```

or

```text
Google Account Already Registered
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

Authentication token or OTP expired.

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

## Email Flow

```text
Username Available
↓
Create Reservation
↓
Enter Email
↓
Send OTP
↓
Verify OTP
↓
Create Approved User Account
↓
Create Public Profile
```

---

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
* Apple Sign-In
* Multi-factor authentication
* Identity linking
* Account recovery
* Security levels
* Device trust systems

These features are outside the scope of V1.
