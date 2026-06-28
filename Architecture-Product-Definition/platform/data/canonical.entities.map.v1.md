# Minime Canonical Entities Map V1

**Status:** Canonical
**Version:** V1
**Platform Service:** Data
**Parent:** `platform/data/data.architecture.specification.v1.md`
**Repository State:** Canonical Repository State V1.0

---

# 1. Purpose

This document defines the canonical entity map for Minime V1.

It identifies the main structured records that Minime needs in order to operate the approved V1 product architecture.

This is not a database schema.

This is not an implementation model.

This is not a table design.

This is a conceptual map of durable data entities, their ownership, their source of truth, their lifecycle expectations, and their relationships.

The goal is simple:

> every meaningful structured record in Minime V1 should have a clear place, owner, and purpose.

---

# 2. Scope

This map covers canonical entities required by Minime V1 across:

* Account (including settings and QR code)
* Connected Accounts
* Profile (profile content, blocks, design, block styling)
* Rendering (including the public profile surface)
* Out Links
* Analytics
* Social Accounts
* Platform Data
* Platform Storage metadata
* Platform Events
* Platform AI suggestions

This document does not define:

* SQL tables
* ORM classes
* migration files
* indexes
* database engine choices
* physical storage layout
* API payloads
* frontend state models

Those belong to future implementation planning.

---

# 3. Entity Map Principles

Every canonical entity in Minime V1 must follow these principles.

---

## 3.1 One Entity, One Purpose

Each entity must represent one clear concept.

An entity should not combine unrelated responsibilities.

For example:

* a Block record represents profile content structure
* an Out Link record represents a trackable outbound route
* an Analytics Event represents something that happened

They should not be merged just because they are related.

---

## 3.2 Identity Is Not Ownership

Every durable entity should have its own record identity.

Ownership is separate.

For example:

```text
block_id = identity of the block
account_id = owner of the block
```

`account_id` must not be used as the primary identity of every owned record.

It identifies the owner.

It does not identify the record itself.

---

## 3.3 Account Is the Ownership Boundary

In Minime V1, account-owned records are owned through `account_id`.

There is no persisted `profile_id` ownership model.

Profile is the public expression of Account-owned data.

Any owned entity must reference `account_id` unless explicitly defined as system-owned.

---

## 3.4 User Is the Source of Truth

User-owned product data becomes canonical only when entered, confirmed, arranged, or approved by the user.

AI, Analytics, external platforms, generated data, and inferred data are never the source of truth.

They may assist, observe, or suggest.

They must not silently become canonical user data.

---

## 3.5 Entity Map Is Not Schema

This file names entities and their relationships.

It does not finalize:

* column names
* field types
* constraints
* foreign keys
* indexes
* database normalization
* storage engines

Implementation may choose suitable physical structures later, as long as the conceptual ownership and lifecycle remain intact.

---

# 4. Entity Categories

Minime V1 entities are grouped into the following categories:

```text
Account Entities
Profile Entities
Connected Account Entities
Out Link Entities
Analytics Entities
Storage Metadata Entities
Event Entities
AI Entities
System Support Entities
```

These categories are for clarity.

They are not required to become database schemas or modules.

---

# 5. Core Entity Overview

The following entities form the canonical data map for V1.

| Entity                     | Category           | Owner                                   | Canonical?                     | Public?           |
| -------------------------- | ------------------ | --------------------------------------- | ------------------------------ | ----------------- |
| Account                    | Account            | System / User-controlled                | Yes                            | Partially         |
| Authentication Identity    | Account            | Account                                 | Yes                            | No                |
| Account Settings           | Account            | Account                                 | Yes                            | No                |
| Profile Content            | Profile            | Account                                 | Yes                            | Partially         |
| Block                      | Profile            | Account                                 | Yes                            | Partially         |
| Block Style Override       | Profile            | Account                                 | Yes                            | No                |
| Appearance Configuration   | Profile            | Account                                 | Yes                            | Partially         |
| Theme Selection            | Profile            | Account                                 | Yes                            | Partially         |
| Connected Account          | Connected Accounts | Account                                 | Yes                            | Yes               |
| Social Platform Definition | System             | System                                  | Yes                            | Yes               |
| Platform Authorization     | Connected Accounts | Account                                 | No                             | No / Future       |
| Out Link                   | Out Links          | Account                                 | Yes                            | Public route only |
| QR Code                    | Account            | Account                                 | Yes                            | Public route only |
| Analytics Event            | Analytics / Events | System, account-scoped where applicable | Yes                            | No                |
| Analytics Aggregate        | Analytics          | System, account-scoped where applicable | Derived                        | No                |
| Stored Asset Metadata      | Storage            | Account or System                       | Yes                            | No                |
| AI Suggestion              | AI                 | Account-scoped                          | Temporary / Optional Canonical | No                |
| Render Object              | Rendering          | Generated                               | No                             | Yes               |
| Public Profile Route       | Rendering          | Account                                 | Yes                            | Yes               |

---

# 6. Account Entity

## Purpose

The Account entity represents the owner boundary for Minime V1.

It is the root entity for user-owned resources.

## Owner

```text
System-created
User-controlled
```

## Source of Truth

The user is the source of truth for account-facing identity decisions.

The system is the source of truth for internal account identity and lifecycle metadata.

## Data Category

```text
User-Owned Data
System-Owned Data
```

## Public Visibility

Partially public.

Some account-derived values may appear publicly through the rendered profile, such as username or display identity.

Internal account metadata must never be public.

## Key Relationships

```text
Account
├── owns Profile Content
├── owns Blocks
├── owns Appearance Configuration
├── owns Social Accounts
├── owns Out Links
├── owns QR Code
├── owns Stored Asset Metadata
├── owns Account Settings
└── scopes Analytics and AI Suggestions
```

## Lifecycle

```text
Created
↓
Active
↓
Updated
↓
Disabled / Deleted
```

Permanent deletion must trigger the account-owned lifecycle rules defined in:

```text
platform/data/data.ownership.and.lifecycle.specification.v1.md
```

---

# 7. Authentication Identity Entity

## Purpose

Authentication Identity represents how an Account signs in or verifies access.

Examples include:

* email authentication
* Google authentication
* OTP verification state where applicable

## Owner

```text
Account
```

## Source of Truth

The user provides or confirms authentication identifiers.

The system owns verification state and security metadata.

## Data Category

```text
User-Owned Data
System-Owned Data
Temporary Data
```

## Public Visibility

Never public.

## Key Relationships

```text
Authentication Identity
└── belongs to Account
```

## Lifecycle

Authentication identity may be:

```text
Created
↓
Verified
↓
Updated
↓
Disabled / Removed
```

Temporary verification state must expire.

---

# 8. Account Settings Entity

## Purpose

Account Settings stores account-level preferences and configuration that are not public profile content.

## Owner

```text
Account
```

## Source of Truth

The user is the source of truth for user-facing settings.

The system is the source of truth for operational settings defaults.

## Data Category

```text
User-Owned Data
System-Owned Data
```

## Public Visibility

Not public unless a Product Domain explicitly exposes a setting through the rendered profile.

## Key Relationships

```text
Account Settings
└── belongs to Account
```

## Lifecycle

Settings are created with the Account and updated over time.

They are deleted or anonymized according to Account deletion rules.

---

# 9. Profile Content Entity

## Purpose

Profile Content represents the account-owned content that may appear on the public Minime profile.

It is not a separate ownership root.

It is an account-owned content structure.

## Owner

```text
Account
```

## Source of Truth

The user.

## Data Category

```text
User-Owned Data
```

## Public Visibility

Partially public after Rendering.

## Key Relationships

```text
Profile Content
├── belongs to Account
├── organizes Blocks
├── references Appearance Configuration
└── may reference Stored Asset Metadata
```

## Lifecycle

```text
Created
↓
Active
↓
Updated
↓
Deleted with Account
```

There is no persisted `profile_id` ownership model.

---

# 10. Block Entity

## Purpose

A Block represents a unit of profile content.

Examples include:

* avatar block
* name block
* bio block
* button block
* social icons block
* image block
* divider block
* title block
* textbox block

## Owner

```text
Account
```

## Source of Truth

The user.

## Data Category

```text
User-Owned Data
System-Owned Metadata
```

## Public Visibility

Partially public after Rendering.

Internal fields such as identity, ownership, and ordering metadata are not public by default.

## Key Relationships

```text
Block
├── belongs to Account
├── may reference Stored Asset Metadata
├── may reference Out Link
├── may have Block Style Override
└── is consumed by Rendering
```

## Lifecycle

```text
Created
↓
Active
↓
Reordered / Updated
↓
Disabled / Deleted
```

Deleted blocks must not appear in rendered output.

---

# 11. Block Style Override Entity

## Purpose

Block Style Override represents account-owned styling decisions applied to a specific Block.

It allows the user to customize how a block appears without changing the block's content meaning.

## Owner

```text
Account
```

## Source of Truth

The user.

## Data Category

```text
User-Owned Data
```

## Public Visibility

Not directly public.

Its resolved effect may appear in rendered output.

## Key Relationships

```text
Block Style Override
├── belongs to Account
├── references Block
└── participates in Rendering
```

## Lifecycle

```text
Created
↓
Active
↓
Updated
↓
Removed
```

If the related Block is deleted, the style override should be deleted or ignored according to lifecycle rules.

---

# 12. Appearance Configuration Entity

## Purpose

Appearance Configuration represents account-owned visual configuration for the public profile.

Examples include:

* theme choice
* colors
* typography-related choices
* background choices
* layout preferences where allowed

## Owner

```text
Account
```

## Source of Truth

The user for selected appearance choices.

The system for default appearance values.

## Data Category

```text
User-Owned Data
System-Owned Defaults
```

## Public Visibility

Partially public through rendered output.

## Key Relationships

```text
Appearance Configuration
├── belongs to Account
├── may reference Theme Selection
├── may reference Stored Asset Metadata
└── is consumed by Rendering
```

## Lifecycle

```text
Created with defaults
↓
Customized
↓
Updated
↓
Deleted with Account
```

---

# 13. Theme Selection Entity

## Purpose

Theme Selection represents the selected visual theme applied to an Account's public profile.

## Owner

```text
Account
```

## Source of Truth

The user selects the active theme.

The system defines available themes.

## Data Category

```text
User-Owned Data
System-Owned Reference Data
```

## Public Visibility

Its visual result is public through Rendering.

Internal theme metadata is not public by default.

## Key Relationships

```text
Theme Selection
├── belongs to Account
├── references System Theme Definition
└── participates in Appearance Configuration
```

## Lifecycle

```text
Default selected
↓
Changed by user
↓
Updated
↓
Deleted with Account
```

---

# 14. Connected Account Entity

## Purpose

Connected Account represents a user-confirmed social platform reference attached to the Account.

It is the canonical storage location for user social link records.

Connected Account records are produced by the Social Accounts domain via a Normalized Social Account Handoff Record and stored canonically by the Connected Accounts domain.

Examples include:

* Instagram username
* TikTok profile URL
* LinkedIn URL
* YouTube URL
* X / Twitter URL

## Owner

```text
Account
```

## Source of Truth

The user.

External platforms are not the source of truth.

AI is not the source of truth.

## Data Category

```text
User-Owned Data
External Reference Data
```

## Public Visibility

Usually public if the user chooses to display it.

## Key Relationships

```text
Connected Account
├── has its own record identity (connected_account_id)
├── belongs to Account
├── is produced by Social Accounts Setup (via Normalized Handoff Record)
├── references Social Platform Definition (via platform field)
└── is rendered through Social Icons Block (read-only consumer)
```

## Required Identity Rule

A Connected Account record must have its own record identity:

```text
connected_account_id
```

It must use:

```text
account_id
```

only as the ownership reference.

It must not use `profile_id`.

It must not use `account_id` as the record's own identity.

## Lifecycle

```text
Created
↓
Active (Exists)
↓
Removed
```

Connected Accounts use Hard Delete.

There is no Hidden, Archived, Suspended, or Draft state.

Removing a Connected Account removes Minime's reference.

It does not delete or modify anything on the external platform.

---

# 15. Social Platform Definition Entity

## Purpose

Social Platform Definition represents a supported social platform inside Minime.

Examples include:

* Instagram
* TikTok
* LinkedIn
* YouTube
* X / Twitter
* Facebook
* Snapchat
* WhatsApp

## Owner

```text
System
```

## Source of Truth

Minime system configuration.

## Data Category

```text
System-Owned Reference Data
```

## Public Visibility

Public as a supported platform option.

## Key Relationships

```text
Social Platform Definition
└── may be referenced by Social Account
```

## Lifecycle

```text
Defined
↓
Active
↓
Deprecated / Removed from selection
```

Deprecating a platform must not silently delete user-owned Social Account references without a defined migration or fallback.

---

# 16. Platform Authorization Entity

## Purpose

Platform Authorization represents a future or optional authorization relationship between Minime and an external account provider.

Platform Authorization does not exist in V1.

It is reserved for future OAuth or external integration capabilities in V2 or later.

Platform Authorization is not the same as Connected Account.

Connected Account (§14) is the canonical social link record owned and confirmed by the user.

Platform Authorization is an access or integration relationship — not a social link.

## Owner

```text
Account
```

## Source of Truth

The user initiates and authorizes the connection.

The external provider may provide technical connection metadata.

## Data Category

```text
User-Owned Data
External Reference Data
System-Owned Operational Data
```

## Public Visibility

Not public by default.

## Key Relationships

```text
Platform Authorization
├── belongs to Account
└── references external provider
```

## Lifecycle

Not applicable in V1.

Reserved for V2+.

---

# 17. Out Link Entity

## Purpose

Out Link represents a trackable outbound route from Minime to an external URL.

It supports link routing, click tracking, and safe public profile behavior.

## Owner

```text
Account
```

## Source of Truth

The user is the source of truth for the destination URL.

The system is the source of truth for routing identifiers.

## Data Category

```text
User-Owned Data
System-Owned Data
External Reference Data
```

## Public Visibility

The public route may be visible.

Internal destination handling and tracking metadata are not directly public.

## Key Relationships

```text
Out Link
├── belongs to Account
├── may be referenced by Button Block
├── may emit Out Link Clicked Event
└── may contribute to Analytics
```

## Lifecycle

```text
Created
↓
Active
↓
Updated
↓
Disabled / Deleted
```

Deleted or disabled Out Links must not route visitors as active links.

---

# 18. QR Code Entity

## Purpose

QR Code represents a generated code that points to an approved Minime route.

Usually this route is the public profile or another approved Minime URL.

## Owner

```text
Account
```

## Source of Truth

The user owns the target decision.

The system generates the QR representation.

## Data Category

```text
Generated Data
System-Owned Data
User-Owned Reference
```

## Public Visibility

The QR output may be public.

Internal QR metadata is not public by default.

## Key Relationships

```text
QR Code
├── belongs to Account
├── targets Public Profile Route or approved route
├── may reference Stored Asset Metadata
└── may emit QR Scanned Event if tracking is enabled
```

## Lifecycle

```text
Generated
↓
Active
↓
Regenerated / Retired
↓
Deleted
```

QR regeneration must not change the user's public profile data.

---

# 19. Public Profile Route Entity

## Purpose

Public Profile Route represents the public address where an Account's rendered profile can be accessed.

Examples include:

```text
/@username
username.minime.ae
custom domain route (future)
```

## Owner

```text
Account
```

## Source of Truth

The user chooses or confirms the public username/handle.

The system owns route resolution rules.

## Data Category

```text
User-Owned Data
System-Owned Routing Data
```

## Public Visibility

Public.

## Key Relationships

```text
Public Profile Route
├── belongs to Account
├── resolves to rendered profile output
└── may emit Profile Viewed Event
```

## Lifecycle

```text
Reserved
↓
Active
↓
Changed / Released / Deleted
```

Route changes must respect username and ownership policies.

---

# 20. Stored Asset Metadata Entity

## Purpose

Stored Asset Metadata represents structured information about a binary asset stored by the Storage Platform Service.

The binary file itself belongs to Storage.

The metadata belongs to Data.

## Owner

```text
Account or System
```

## Source of Truth

The system is the source of truth for technical asset metadata.

The user is the source of truth for whether the asset should be used as part of their profile.

## Data Category

```text
System-Owned Data
Generated Data
User-Owned Reference
```

## Public Visibility

Not public by default.

The served asset URL may become public through Rendering or Public Profile serving.

## Key Relationships

```text
Stored Asset Metadata
├── may belong to Account
├── may be referenced by Block
├── may be referenced by Appearance Configuration
├── may be referenced by QR Code
└── points to Storage-managed binary asset
```

## Lifecycle

```text
Uploaded / Generated
↓
Processing / Active
↓
Replaced / Unused
↓
Deleted / Cleaned up
```

Orphaned asset metadata must be handled by Storage and Data lifecycle policies.

---

# 21. Analytics Event Entity

## Purpose

Analytics Event represents something that happened.

Examples include:

* profile viewed
* out link clicked
* QR scanned

## Owner

```text
System
Account-scoped where applicable
```

## Source of Truth

The event-producing system.

## Data Category

```text
Event Data
System-Owned Data
Derived / Observational Data
```

## Public Visibility

Not public.

Aggregated analytics may be shown to the owning Account.

## Key Relationships

```text
Analytics Event
├── may reference Account
├── may reference Public Profile Route
├── may reference Out Link
├── may reference QR Code
└── may contribute to Analytics Aggregate
```

## Lifecycle

```text
Emitted
↓
Accepted
↓
Stored / Aggregated
↓
Expired / Deleted / Aggregated-only
```

Events describe what happened.

They do not command behavior.

---

# 22. Analytics Aggregate Entity

## Purpose

Analytics Aggregate represents summarized analytics derived from Events.

Examples include:

* total views
* total clicks
* clicks per link
* views over time
* device category totals if supported

## Owner

```text
System
Account-scoped where applicable
```

## Source of Truth

Underlying Events or accepted analytics processing.

## Data Category

```text
Derived Data
System-Owned Data
```

## Public Visibility

Not public to visitors.

Visible only to the owning Account where Product Domain rules allow.

## Key Relationships

```text
Analytics Aggregate
├── derived from Analytics Events
├── scoped to Account where applicable
└── consumed by Analytics reporting
```

## Lifecycle

```text
Generated
↓
Updated
↓
Refreshed / Recalculated
↓
Expired / Deleted
```

Aggregates should not replace canonical product data.

---

# 23. AI Suggestion Entity

## Purpose

AI Suggestion represents an AI-generated recommendation before the user approves it.

Examples include:

* suggested bio text
* suggested username
* suggested button title
* suggested profile improvement
* onboarding suggestion

## Owner

```text
Account-scoped
System-generated
User-approved only if accepted
```

## Source of Truth

AI is the source of the suggestion.

The user is the source of truth for whether it becomes product data.

## Data Category

```text
AI Suggestion Data
Generated Data
Temporary Data
```

## Public Visibility

Never public unless accepted by the user and converted into Product Domain data.

## Key Relationships

```text
AI Suggestion
├── may belong to Account
├── may reference suggested target domain
├── may be accepted, edited, rejected, or expired
└── may become User-Owned Data only through approval
```

## Lifecycle

```text
Generated
↓
Presented
↓
Accepted / Edited / Rejected / Expired
```

Rejected suggestions must not become canonical user data.

---

# 24. Render Object Entity

## Purpose

Render Object represents prepared visitor-safe output generated from approved product data.

It is not a canonical source entity.

It is a read-safe representation.

## Owner

```text
Generated
```

## Source of Truth

Canonical Product Domain data.

## Data Category

```text
Generated Data
Derived Data
Read Model
```

## Public Visibility

Yes.

## Key Relationships

```text
Render Object
├── generated from Account-owned product data
├── consumed by Public Profile
└── may become stale and be regenerated
```

## Lifecycle

```text
Generated
↓
Served
↓
Invalidated / Regenerated
```

Render Objects must not become the source of truth.

Renderer never owns data.

---

# 25. System Theme Definition Entity

## Purpose

System Theme Definition represents a theme made available by Minime.

It is system-owned reference data.

## Owner

```text
System
```

## Source of Truth

Minime.

## Data Category

```text
System-Owned Reference Data
```

## Public Visibility

Public as a selectable visual option.

## Key Relationships

```text
System Theme Definition
└── may be referenced by Theme Selection
```

## Lifecycle

```text
Defined
↓
Available
↓
Deprecated / Removed
```

Removing or changing a theme must respect existing Theme Selection behavior.

---

# 26. Reserved Username Entity

## Purpose

Reserved Username represents a username that cannot be claimed or is temporarily held.

Examples include:

* system-reserved usernames
* blocked terms
* temporary claim reservations
* protected routes

## Owner

```text
System
```

## Source of Truth

Minime username policy.

## Data Category

```text
System-Owned Data
Temporary Data where applicable
```

## Public Visibility

Not public as internal policy data.

The result may be visible as an availability response.

## Key Relationships

```text
Reserved Username
└── participates in Account username claim flow
```

## Lifecycle

```text
Defined / Reserved
↓
Active
↓
Expired / Released where applicable
```

Reserved usernames must not be treated as user-owned accounts until successfully claimed.

---

# 27. Entity Relationship Overview

```text
Account
├── Authentication Identity
├── Account Settings
├── Profile Content
│   └── Blocks
│       ├── Block Style Overrides
│       ├── Stored Asset Metadata
│       └── Out Links
├── Appearance Configuration
│   ├── Theme Selection
│   └── Stored Asset Metadata
├── Connected Accounts
├── Out Links
│   └── Analytics Events
├── QR Codes
│   ├── Stored Asset Metadata
│   └── Analytics Events
├── Public Profile Route
│   └── Analytics Events
├── AI Suggestions
└── Analytics Aggregates
```

Supporting system-owned reference entities:

```text
System
├── Social Platform Definitions
├── System Theme Definitions
└── Reserved Usernames
```

Generated / derived entities:

```text
Canonical Product Data
├── Render Objects
├── Analytics Aggregates
└── AI Suggestions
```

---

# 28. Entities That Must Not Exist in V1

The following must not be introduced as canonical ownership entities in V1:

```text
Profile
Workspace
Organization
Team
Visitor
Audience Member
External Social Profile
Discovery Result
AI Memory Profile
Personal Brand Score
```

Explanation:

* `Profile` is not a persisted owner entity.
* `Workspace`, `Organization`, and `Team` belong to future collaboration or multi-account models.
* `Visitor` is not a canonical user/profile entity in V1.
* `External Social Profile` would incorrectly make external platforms part of Minime's truth model.
* `Discovery Result` would reintroduce the retired Discovery Engine.
* `AI Memory Profile` and `Personal Brand Score` are not approved V1 entities.

---

# 29. Entity Naming Rules

Entity names should be clear and singular.

Record identity names should identify the record itself.

Examples:

```text
account_id
block_id
connected_account_id
out_link_id
qr_code_id
asset_id
event_id
ai_suggestion_id
```

Ownership references should use:

```text
account_id
```

Do not use:

```text
profile_id
```

as an ownership reference in V1.

Do not overload `account_id` to mean both owner and record identity.

---

# 30. Canonical vs Non-Canonical Outputs

Some data structures are useful but not canonical.

Examples:

* Render Objects
* temporary previews
* cached public output
* AI drafts before approval
* analytics summaries
* resolved style output

These may be stored or generated depending on implementation.

But they must not become the source of truth.

If they conflict with canonical Product Domain data, canonical Product Domain data wins.

---

# 31. Final Canon

Minime V1 has one simple data ownership root:

```text
Account
```

Most user-owned product entities belong to Account.

System reference entities support the product.

Generated and derived entities assist the product.

External references are never external truth.

AI suggestions are never user truth until approved.

Render Objects are never source records.

Events describe what happened.

Analytics aggregates observations.

There is no persisted `Profile` ownership model in V1.

There is no `profile_id` ownership model in V1.

Every durable entity must have a clear identity, a clear owner, a clear source of truth, and a clear lifecycle.

This is the canonical entity map for Minime V1.
