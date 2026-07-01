# Minime Account Management System Specification V1

## Status

Approved

---

# Purpose

> The Account Management System is part of the Account domain. Account Management is not a standalone Product Domain.

The Minime Account Management System is responsible for managing an approved Minime account after account creation.

Its responsibility begins immediately after an Approved User Account exists.

The system persists the user-provided data in:

* Connected Accounts
* Authentication Identity
* Profile Information
* Account Settings

The user remains the source of truth. The system stores the canonical representation of what the user intentionally provided or confirmed; it never originates or replaces user intent automatically.

The Account Management System is independent from:

* Account Claim System
* Social Accounts Setup
* Future Profile Import Engine

---

# Scope

The Account Management System is responsible for:

* Connected account management
* Profile editing
* Authentication identity management
* Account settings management
* Connected account validation
* Connected account lifecycle management

The Account Management System is not responsible for:

* Username creation
* Username ownership
* Username changes
* Public profile URL changes
* Social account collection and normalization
* Social account ownership verification
* Profile data import
* Analytics
* Historical profile versions

---

# Core Principles

## Current State Only

Minime stores only the current version of account data.

The system does not maintain:

* Profile version history
* User-visible change history
* Restore points
* Rollback functionality

New values replace old values.

---

## Last Write Wins

When a user updates data:

```text
Old Value
↓
New Value
```

The new value replaces the old value.

The previous value is discarded.

---

## User Is The Source Of Truth

Minime never decides which connected accounts belong to a user.

The user decides:

* Which accounts to add
* Which accounts to remove
* Which accounts to edit

The system only validates the format of each connected account record. It never checks whether the account exists on the external platform.

---

## Unlimited Connected Accounts

Users may connect any number of accounts.

There is no platform limit.

Example:

```text
Instagram
├── @ahmed
├── @ahmedagency
└── @ahmedstore

YouTube
├── @ahmed
└── @ahmedpodcast
```

All are valid.

---

## Platform Independence

Connected accounts are independent.

A change to one connected account must never affect another account.

---

## No Status Model

Connected Accounts do not have statuses.

There is no:

* Pending
* Active
* Inactive
* Verified
* Imported
* Archived

A connected account either exists or does not exist.

---

# Connected Account Model

## Definition

A Connected Account represents a platform account that the user has chosen to associate with their Minime profile.

A Connected Account is a lightweight entity.

It is not a profile import record.

It is not an analytics record.

It is not an ownership verification record.

---

## Required Fields

Every Connected Account contains:

```text
connected_account_id
account_id
platform
username
url
```

---

## Example

```json
{
  "connected_account_id": "acc_01JXXXX",
  "account_id": "acc_01JXXXX",
  "platform": "instagram",
  "username": "ahmedofficial",
  "url": "https://www.instagram.com/ahmedofficial/"
}
```

---

## Out Of Scope

Connected Accounts must not store:

* Followers
* Following
* Verification Status
* Bio
* Avatar
* Profile Metrics
* Analytics Data
* Imported Profile Data

These belong to future systems.

---

# Connected Account Operations

## Creation Sources

Connected Accounts may be created through:

### Social Accounts Setup

```text
Normalized Social Account Record
↓
Save
↓
Create Connected Account
```

Account creation occurs immediately.

No intermediate review state exists.

---

### Manual Addition

```text
Platform
+
Username
↓
Validation
↓
Create Connected Account
```

---

## Manual Input Format

Users manually add accounts using:

```text
Platform
+
Username
```

Example:

```text
Instagram
ahmedofficial
```

The system generates the canonical URL using the platform rules.

Example:

```text
https://www.instagram.com/ahmedofficial/
```

---

## Validation Requirement

All manually added accounts must be validated before creation.

Validation uses the platform rules and checks format only — per `connected.accounts.specification.v1.md`, the system never checks whether the account exists on the external platform. There is no external network call of any kind during Connected Accounts creation.

If the format is invalid:

```text
Creation Rejected
```

The account is not stored.

---

## Duplicate Prevention

Duplicate Connected Accounts are not allowed.

Example:

```text
Instagram
@ahmedofficial
```

cannot be added twice by the same user.

---

## Editing

Connected Accounts are editable.

Users may update:

```text
username
```

The updated value replaces the existing value.

The connected record remains the same record.

Example:

```text
connected_account_id = acc_01JXXXX
```

does not change.

The canonical URL is always generated by the platform rules.

Users cannot edit URLs directly.

---

## Platform Changes

Platform changes are not supported.

Example:

```text
Instagram
↓
TikTok
```

is not an edit.

It is a different account.

Users must:

```text
Remove Existing Account
↓
Create New Account
```

---

## Removal

Connected Account removal uses:

```text
Hard Delete
```

Example:

```text
Remove Account
↓
Delete Connected Account
```

The account is permanently removed.

No account archive exists.

No restore functionality exists.

---

# Profile Editing Rules

## Editable Fields

All profile fields are editable except:

```text
Username
Public Profile URL
```

Examples of editable data:

* Display Name
* Bio
* Avatar
* Connected Accounts
* Contact Information
* Future Profile Fields

---

## Username

Usernames are not editable in V1.

Username ownership remains governed by Account Claim System.

---

## Public Profile URL

Public profile URLs are derived from usernames.

Because usernames cannot change:

```text
Public Profile URL
```

cannot change.

---

## Update Strategy

All profile updates follow:

```text
Last Write Wins
```

The newest value replaces the previous value.

No profile version history exists.

No draft system exists.

No restore system exists.

---

# Authentication Identity Management

## Authentication Model

Minime V1 uses:

```text
External Identity Provider
```

Minime never authenticates users directly. Authentication is provided exclusively by supported external identity providers, modeled through provider adapters (see `authentication.policy.v1.md` — "Provider Adapter Architecture"). `provider` is request data, never API route structure.

Supported providers (V1):

* Google Sign-In
* Apple Sign-In

Passwords, email OTP, and all other direct authentication methods are not supported.

---

## Linked Authentication Identities

An account may have one or both supported providers linked simultaneously.

Allowed providers:

```text
google
apple
```

An account is never required to link both. One linked provider is sufficient at all times.

---

## Primary Provider

If both Google and Apple are linked, either may be designated `primary_provider` — a UI/default-display preference only.

* `primary_provider` must never determine ownership.
* `primary_provider` must never determine authorization.
* `account_id` + the linked `AuthenticationIdentity` binding (`provider` + `provider_subject`) determines ownership — never the primary flag.
* Removing or changing `primary_provider` must never affect authentication validity for any linked identity.
* The last remaining linked provider cannot be removed, regardless of whether it is marked primary.

---

## Minimum Requirement

Every account must always have:

```text
At Least One Authentication Provider Linked
```

The last remaining authentication provider cannot be removed. Unlinking must be rejected if it would leave the account with zero linked providers.

---

## Linking and Unlinking a Provider

Linking adds a new authentication identity to the account; it requires the user to already be logged in and to explicitly start provider linking, completing a full OAuth verification flow with the new provider. It is subject to the same duplicate-identity check used during registration: a `(provider, provider_subject)` pair already bound to a different account must never be linked to a second account.

Unlinking removes an authentication identity record, identified by `auth_identity_id`. Unlinking never removes the account and never affects any other linked identity or any Product Data.

There is no "replace an identity" operation. `provider_subject` is immutable once an identity record is created; an identity is linked or unlinked, never edited or replaced. There is no email-based identity change and no automatic account merge of any kind, because email is never the authentication identity in V1. The full rules are defined in `authentication.policy.v1.md` — "Identity Merge Policy".

---

# Session Policy

**Governing decision:** This section was updated to adopt the session model already approved and implemented in `implementation/08-security-model.md`. The obsolete flat "7-day session, no session management" model previously described here was never implemented and directly contradicted the approved implementation. See `ARCHITECTURE_PR_APPROVAL_DECISIONS.md` — APD-012.

## Session Creation

Successful provider authentication creates a user session, composed of a short-lived access token and a longer-lived, rotatable refresh token.

Example:

```text
Provider Sign-In (Google or Apple)
↓
Authenticated
↓
Session Record Created (Session.refresh_token_hash)
↓
Access Token (JWT) + Refresh Token Issued
```

---

## Token Model

* **Access token:** short-lived JWT (15 minutes), never persisted, carries `{sub: account_id, session_id, iat, exp, iss, aud}`.
* **Refresh token:** opaque value, rotated on every use (old hash invalidated, new hash stored), 30-day sliding lifetime measured from last use. Only its hash is stored (`Session.refresh_token_hash`); the plain value is never persisted.

Full token rules: `implementation/08-security-model.md` — "Token Rules."

---

## Session Expiry

A refresh token is rejected once its Session record is expired or revoked. When the refresh token is rejected, the user must re-authenticate using their registered provider to obtain a new session.

```text
Refresh Token Expired Or Revoked
↓
Provider Sign-In (Google or Apple)
↓
Authenticate
↓
New Session
```

---

## Session Management, Device Lists, and Logout All Devices

V1 supports session management as a first-class capability:

* **Session list:** an account owner may list their own active, non-expired, non-revoked sessions (`GET /api/v1/auth/sessions`). The response never exposes refresh tokens or internal session identifiers beyond what is needed to distinguish sessions.
* **Logout (current session only):** `POST /api/v1/auth/logout` revokes the current session's refresh token.
* **Logout all devices:** `POST /api/v1/auth/logout-all` revokes every active, non-expired session for the account in one call — this is the "Logout All Devices" capability.
* **Trusted devices:** V1 does not label or distinguish sessions by device trust. Every session is equivalent; there is no "trusted device" concept.

Full session and logout API contracts: `implementation/05-api-contracts.md` — Part 2. Full session security rules (per-request account status check, revocation triggers): `implementation/08-security-model.md` — "Session Security."

---

# Social Accounts Integration

## Social Accounts Relationship

Social Accounts Setup is not part of Account Management.

Social Accounts Setup remains a separate domain that collects, normalizes, and generates URLs for user-provided social account identifiers.

---

## Save Flow

```text
Normalized Social Account Record
↓
Save
↓
Create Connected Account
```

No additional review process exists.

---

## Existing Accounts

If Social Accounts Setup produces a record that already exists in Account Management:

The account remains visible.

The result should be marked:

```text
Already Added
```

The user remains responsible for all decisions.

---

# Audit Logging

Administrative audit logging is defined separately in:

```text
audit.logging.policy.v1.md
```

Account Management may emit audit events but does not define audit retention, storage, access controls, or operational policies.

---

# Related Documents

```text
connected.accounts.specification.v1.md
profile.content.specification.v1.md
audit.logging.policy.v1.md
authentication.policy.v1.md
minime.account.claim.system.specification.v1.md
```
