# Account Model Specification V1

## Status

Approved

---

# Purpose

> The Account domain is a single domain that owns account identity, account management, settings, and the QR code entry point. Account Management, Settings, and QR Code are implementation areas within the Account domain, not standalone Product Domains.

This document defines the Account model used by Minime V1.

The Account is the single identity root of Minime.

In V1, Minime does not separate:

```text
Account
User
Profile
Public Profile Owner
```

These concepts refer to the same architectural unit.

---

# Core Decision

In Minime V1:

```text
Account = User = Profile
```

There is no separate Profile entity.

There is no separate User entity.

There is no separate Profile ID.

The Account is the single source of identity, ownership, public presence, and profile ownership.

---

# Definition

An Account represents one claimed Minime identity.

Each Account owns:

```text
Account Identity
Username
Public URL
Authentication Identity
Public Profile Content
Connected Accounts
Blocks
Settings
Account Status
```

The public profile is the public-facing representation of the Account.

---

# Account Identity

## Internal Account ID

Every Account has one internal identifier.

Example:

```text
account_id = usr_01JXXXX
```

The internal Account ID is:

```text
Globally Unique
Immutable
Permanent
System-Owned
```

The internal Account ID must never change.

---

## Username

Every Account has one public username.

Example:

```text
username = ahmed
```

The username is:

```text
Public
Unique
User-Chosen
Claimed During Account Creation
Used For Public Routing
```

The username represents the public Minime identity.

---

# Identity Relationship

In V1, the following are treated as one architectural identity:

```text
account_id
username
account
user
profile
```

This does not mean they are the same database field.

It means they represent the same product unit.

There must not be a separate model called:

```text
User
Profile
PublicProfile
```

unless explicitly introduced in a future version.

---

# Public Profile

A Profile in Minime V1 is not a separate entity.

A Profile means:

```text
The public-facing representation of an Account.
```

When a visitor opens:

```text
https://minime.ae/{username}
```

the system resolves the username to one Account.

The rendered page is the public profile view of that Account.

---

# Public URL

Each Account owns exactly one public URL.

Format:

```text
https://minime.ae/{username}
```

The public URL is derived from the username.

It is not stored as an independent editable value.

Because usernames are not editable in V1, public URLs are not editable in V1.

---

# Ownership Model

In Minime V1:

```text
One Account
=
One Username
=
One Public URL
=
One Public Profile
```

Not supported:

```text
One Account → Multiple Profiles
One User → Multiple Accounts
One Profile → Multiple Owners
Multiple Public URLs Per Account
```

---

# Account Responsibilities

The Account is responsible for owning references to:

```text
Username
Authentication Identity
Profile Content
Connected Accounts
Theme
Blocks
Settings
Status
```

The Account is the ownership root for all account-owned data.

---

# Account Does Not Own

The Account model does not own implementation details of:

```text
Authentication Flow
Username Validation Rules
Connected Account Validation
Social Accounts Setup
Theme Definitions
Rendering
Style Resolution
Analytics Events
Out Link Tracking
Cache Policy
Public Request Handling
```

These are defined in their own systems.

The Account only owns the identity and references required to connect these systems.

---

# Account Model

## Required Fields

```ts
type Account = {
  id: string;
  username: string;
  status: AccountStatus;
  created_at: string;
  updated_at: string;
};
```

---

# Field Definitions

## id

The internal Account identifier.

Example:

```text
usr_01JXXXX
```

Rules:

```text
Required
Globally Unique
Immutable
System Generated
Never Publicly Editable
```

---

## username

The public Minime username.

Example:

```text
ahmed
```

Rules:

```text
Required
Globally Unique
User Claimed
Publicly Visible
Used For Routing
Not Editable In V1
```

Username rules are defined separately in:

```text
username.policy.v1.md
```

---

## status

Defines the system-level state of the Account.

Allowed V1 statuses:

```text
active
suspended
deleted
```

---

## created_at

Timestamp of Account creation.

---

## updated_at

Timestamp of last Account-level update.

---

# Account Status

## active

The Account exists and may be used normally.

An active Account may:

```text
Authenticate
Edit account-owned data
Manage connected accounts
Manage profile content
Render a public profile
Track out links
Generate analytics
```

---

## suspended

The Account exists but is restricted by the system.

A suspended Account may not render a public profile.

A suspended Account may not authenticate. Existing sessions are invalidated on suspension. New session creation is rejected.

A suspended Account may not be treated as deleted.

The username remains reserved.

Suspension does not result in data deletion. All account data is preserved.

Suspension is reversible. A suspended account may be restored to `active` status.

Suspension is an administrative action. No user may self-suspend their own account. No automated system may suspend accounts in V1. The criteria and process for suspension are operational policy, not product architecture. The architecture defines only the behavioral consequences of the `suspended` status.

---

## deleted

The Account has been deleted by the account owner.

The complete deletion lifecycle is defined in:

```text
account/account.deletion.policy.v1.md
```

A deleted Account must not render a public profile.

A deleted Account must not allow authentication or account access.

The account identifier is retained permanently for audit log referential integrity.

The username is permanently reserved and cannot be claimed by another user in V1.

---

# Profile Content Relationship

Profile Content belongs directly to the Account.

There is no `profile_id`.

Profile Content should reference:

```text
account_id
```

not:

```text
profile_id
```

Profile Content owns account-owned public content such as:

```text
Display Name
Avatar
Bio
Contact Information
Block Order
Block References
```

Profile Content is a module, not an entity root.

---

# Blocks Relationship

Blocks belong directly to the Account-owned public profile content.

Every Block must reference the owning Account.

Recommended field:

```text
account_id
```

Not recommended:

```text
profile_id
```

Because Minime V1 does not have a separate Profile entity.

---

# Appearance Relationship

The Account does not own Appearance State directly.

Appearance State is stored as a section within the Profile Content record, identified by `account_id`.

The Account does not own Theme Definitions.

Theme Definitions are immutable catalog assets managed by the Theme Library, which is a subsystem of the Appearance system.

The active theme reference (`selected_theme_id`) and any user customization values are part of the Appearance State within the Profile Content record.

Theme definitions, defaults, constraints, and the Theme Catalog are owned by the Appearance system, not by the Account.

---

# Connected Accounts Relationship

Connected Accounts belong to the Account.

Each Connected Account references the owning Account.

Connected Accounts are external platform references approved or added by the user.

They do not define the Minime Account identity.

---

# Authentication Relationship

Authentication belongs to the Account.

An Account must have at least one valid authentication method according to the Authentication Policy.

Authentication flow and verification rules are not defined by this file.

---

# Public Routing Relationship

Public routing resolves:

```text
username
↓
Account
↓
Public Profile View
```

There is no routing step that resolves a separate Profile entity.

Invalid routing examples:

```text
username
↓
Profile
↓
Account
```

or:

```text
username
↓
User
↓
Profile
```

The correct V1 model is:

```text
username
↓
Account
```

---

# Visibility And Publishing

In V1, the Account model does not introduce a separate publish model.

Public visibility rules belong to the Public Profile system.

Block visibility rules belong to the Block system.

Account status may prevent public rendering, but it does not replace public visibility policy.

---

# Important Constraints

## No Separate Profile Model

Minime V1 must not introduce:

```text
profile_id
profile_owner_id
profile_status
profile_entity
profiles_table
```

unless a future version explicitly supports multiple profiles per account.

---

## No Multiple Profiles

V1 does not support:

```text
Multiple profiles per account
Sub-profiles
Brand profiles
Team profiles
Client profiles
```

---

## No Multi-Page Structure

V1 does not support:

```text
Sub Pages
Nested Pages
Multi Page Profiles
```

A public Account renders one public page.

---

## Username Is Not A Separate Account

The username is part of the Account identity.

The username must not be modeled as a separate owner of content.

---

# Canonical Account Shape

```ts
type Account = {
  id: string;
  username: string;
  status: "active" | "suspended" | "deleted";
  created_at: string;
  updated_at: string;
};
```

---

# Canonical Ownership Rule

All account-owned systems should reference:

```text
account_id
```

They should not reference:

```text
profile_id
```

because Profile is not an independent entity in Minime V1.

---

# System Invariant

For Minime V1, this must always remain true:

```text
One Account
=
One Username
=
One Public URL
=
One Public Profile
```

The Account is the aggregate root.

The public profile is the Account rendered publicly.

---

# Related Documents

```text
minime.account.claim.system.specification.v1.md
minime.account.management.system.specification.v1.md
authentication.policy.v1.md
username.policy.v1.md
account.deletion.policy.v1.md
connected.accounts.specification.v1.md
profile.content.specification.v1.md
block.system.specification.v1.md
public-profile.system.specification.v1.md
```
