# Audit Logging Policy V1

## Status

Approved

---

# Purpose

This document defines the audit logging rules used by Minime.

Audit Logging exists to support:

* Security Investigation
* Platform Operations
* Customer Support
* Issue Resolution
* Administrative Oversight

Audit Logs are operational records.

Audit Logs are not part of the user-facing product.

---

# Scope

This policy applies to:

* Account Management
* Authentication
* Connected Accounts
* Profile Content
* Administrative Actions

This policy does not apply to:

* Analytics
* Social Accounts Setup
* User Activity Tracking
* Marketing Events

---

# Core Principles

## Operational Purpose Only

Audit Logs exist exclusively for operational and security purposes.

Audit Logs are not product features.

Users cannot interact with audit logs.

---

## Not User Facing

Users cannot:

* View audit logs
* Search audit logs
* Export audit logs
* Restore previous values
* Access historical changes

Audit Logs are internal only.

---

## Current State Remains Authoritative

The active account data remains the source of truth.

Audit Logs do not affect:

* Profile Content
* Connected Accounts
* Authentication Settings

Audit Logs are historical records only.

---

## No Rollback Functionality

Audit Logs are not a versioning system.

V1 does not support:

* Rollback
* Restore
* Undo
* Revision Recovery

---

# Audit Event Model

## Required Fields

Every audit event contains:

```text
log_id
timestamp
account_id
action
entity_type
entity_id
old_value
new_value
metadata
```

---

## Example

```json
{
  "log_id": "log_01JXXXX",
  "timestamp": "2026-06-23T12:00:00Z",
  "account_id": "acc_01JXXXX",
  "action": "profile_updated",
  "entity_type": "profile",
  "entity_id": "acc_01JXXXX",
  "old_value": {
    "display_name": "Ahmed"
  },
  "new_value": {
    "display_name": "Ahmed Salem"
  }
}
```

---

# Field Definitions

## log_id

Audit event identifier.

Requirements:

```text
Globally Unique
Immutable
Permanent
```

---

## timestamp

UTC timestamp representing when the event occurred.

---

## account_id

Account associated with the event.

Identifies the Account that performed the operation.

---

## action

Describes the operation that occurred.

Examples:

```text
profile_updated
account_added
account_removed
identity_changed
```

---

## entity_type

Target entity affected by the operation.

Examples:

```text
profile
connected_account
authentication_identity
page
block
```

---

## entity_id

Identifier of the affected entity.

---

## old_value

State before the change.

---

## new_value

State after the change.

---

## metadata

Optional operational metadata.

Examples:

```text
ip_address
user_agent
admin_id
reason
```

Implementation may extend this structure.

---

# Logged Events

**Governing note:** This catalog was corrected during the Phase A architecture-synchronization audit (G-04) to remove events for concepts that do not exist in V1, per the "Operational Purpose Only" principle above. `account_updated` (as a single blanket event), `recovery_email_changed`, `page_*`, and `block_*` events were previously listed here despite no `Recovery Email` concept, no `Page` entity distinct from the single public profile, and no per-block audit event existing anywhere else in the approved V1 architecture or implementation. See `implementation/03-canonical-data-model.md` — `AuditLog` — "Logged Event Catalog," which already reflected this correction; this document is now updated to match it directly rather than only in the implementation layer.

## Account Events

The system should log:

```text
account_created
account_deleted
```

`account_updated` is not logged as a single blanket event in V1 — `Account.settings` mutations are not individually itemized, since `settings` in V1 has exactly one field (`gtm_container_id`); see `account/account.model.specification.v1.md` and `implementation/03-canonical-data-model.md` — `Account.settings`.

---

## Authentication Events

The system should log:

```text
authentication_method_added
authentication_method_removed
primary_identity_changed
```

`recovery_email_changed` is not a V1 event. There is no Recovery Email concept in V1 — authentication is provider-based only (`account/authentication.policy.v1.md`).

---

## Connected Account Events

The system should log:

```text
connected_account_added
connected_account_updated
connected_account_removed
connected_account_reordered
```

---

## Profile Events

The system should log:

```text
display_name_updated
avatar_updated
bio_updated
contact_information_updated
```

Block create/update/delete/reorder events (`block_created`, `block_updated`, `block_deleted`, `block_reordered`) are **not** logged individually in V1. There is no Page entity distinct from the single public profile in V1 (no `page_created`/`page_updated`/`page_deleted`/`page_reordered` events exist). Block-level changes are Product Data changes already fully covered by the `Block` row's own `updated_at`/`sort_order` history, not audit-logged individually — this is consistent with the "Operational Purpose Only" principle: audit logging scope in V1 is limited to identity, access, and account-lifecycle events, not general content editing history.

---

## Administrative Events

The system should log:

```text
admin_action
support_action
manual_intervention
```

when performed by authorized staff.

---

# Value Capture Rules

## Before And After State

When practical, audit events should contain:

```text
old_value
new_value
```

for the affected data.

---

## No Partial Ambiguity

Audit entries should contain enough information to understand:

```text
What Changed
```

without requiring reconstruction from multiple records.

---

# Social Accounts Relationship

Social Accounts Setup is excluded from audit logging.

Do not store:

* Social Accounts Setup sessions
* Smart Mode input
* Manual Mode input
* Normalization or URL generation history

Social Accounts Setup input is transient and is not retained as an audit trail. Only the resulting saved social account records are subject to the normal Connected Accounts data rules.

---

# Retention Policy

## Retention Period

Audit logs must be retained for:

```text
12 Months
```

---

## Expiration

After the retention period:

```text
Delete Permanently
```

No archival requirement exists in V1.

---

# Access Control

## Authorized Access Only

Audit Logs may only be accessed by:

* Platform Operations
* Security Personnel
* Authorized Support Staff

---

## User Access

Users have no direct access to audit logs.

---

# Security Requirements

Audit Logs should be protected against:

* Unauthorized Access
* Unauthorized Modification
* Unauthorized Deletion

Audit records should be treated as sensitive operational data.

---

# Out Of Scope

This policy does not define:

* Analytics
* Product Metrics
* Event Tracking
* User Behavior Analysis
* Fraud Detection Systems
* Security Scoring

These belong to separate systems.

---

# Related Documents

```text
minime.account.management.system.specification.v1.md
connected.accounts.specification.v1.md
profile.content.specification.v1.md
authentication.policy.v1.md
```

---

# Final Definition

Audit Logging V1 is an internal operational logging system that records meaningful account changes, authentication changes, content changes, and administrative actions for security, support, and operational purposes while remaining completely invisible to end users.
