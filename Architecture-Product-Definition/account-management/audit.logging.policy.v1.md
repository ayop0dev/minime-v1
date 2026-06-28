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

## Account Events

The system should log:

```text
account_created
account_updated
account_deleted
```

where applicable.

---

## Authentication Events

The system should log:

```text
authentication_method_added
authentication_method_removed
primary_identity_changed
recovery_email_changed
```

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

---

## Page Events

The system should log:

```text
page_created
page_updated
page_deleted
page_reordered
```

---

## Block Events

The system should log:

```text
block_created
block_updated
block_deleted
block_reordered
```

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
