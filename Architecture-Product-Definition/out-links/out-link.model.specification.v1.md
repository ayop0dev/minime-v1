# Out Link Model Specification V1

## Status

Approved

---

# Purpose

The Out Link Model defines the data structure used by the Out Link System.

The model represents a single Clickable State and provides all information required for:

```text
Resolution
Redirects
Tracking
Analytics
Lifecycle Management
```

---

# Core Principle

```text
One Out Link
=
One Clickable State
```

The model must never represent:

```text
An Entire Profile
A Block Collection
A Theme
A Presentation Layer
```

---

# Entity Name

```text
Out Link
```

---

# Primary Key

Internal identifier:

```text
id
```

Requirements:

```text
Unique
System Generated
Immutable
```

Example:

```text
ol_01JX8N7W4T3M9K2R5A6B
```

Implementation format is system-defined.

Public routes must never expose this value.

---

# Public Identifier

Public identifier:

```text
public_id
```

Used for:

```text
Routing
Resolution
Public Access
```

Example:

```text
k7m2x9qp
```

Public route:

```text
/out/k7m2x9qp
```

Requirements:

```text
Unique
Immutable
Non Sequential
URL Safe
```

---

# Account Reference

Every Out Link belongs to exactly one Account.

Field:

```text
account_id
```

Purpose:

```text
Ownership
Resolution
Analytics Grouping
Lifecycle Control
```

---

# Source Block Reference

The Block that generated the Out Link.

Field:

```text
block_id
```

Purpose:

```text
Traceability
Analytics Context
Content History
```

Rules:

```text
Nullable = No
Immutable = Yes
```

---

# Source Block Type

Field:

```text
block_type
```

Examples:

```text
button
social
```

Purpose:

```text
Analytics
Filtering
Reporting
```

---

# Destination URL

Field:

```text
destination_url
```

Represents:

```text
Resolved External Destination
```

Examples:

```text
https://google.com
https://youtube.com/@user
https://wa.me/...
```

Rules:

```text
Required
Absolute URL
Stored As Snapshot
```

---

# Clickable Label Snapshot

Field:

```text
label_snapshot
```

Represents the text shown to the visitor when the Out Link was created.

Examples:

```text
Book Consultation
Visit Website
Watch On YouTube
```

Purpose:

```text
Historical Analytics
Future Comparison
```

---

# Source Snapshot

Field:

```text
source_snapshot
```

Purpose:

```text
Store Original Clickable State
```

Example:

```json
{
  "type": "button",
  "title": "Book Consultation",
  "url": "https://wa.me/..."
}
```

Rules:

```text
Immutable
Stored At Creation Time
```

---

# Status

Field:

```text
status
```

Allowed Values:

```text
created
active
archived
```

Notes:

```text
Purged records do not exist.
```

After purge:

```text
Record Deleted
```

---

# Archive Date

Field:

```text
archived_at
```

Purpose:

```text
Retention Management
```

Rules:

```text
Nullable While Active
Required When Archived
```

---

# Created Timestamp

Field:

```text
created_at
```

Purpose:

```text
Auditing
Analytics
Lifecycle
```

Immutable.

---

# Updated Timestamp

Field:

```text
updated_at
```

Purpose:

```text
Operational Tracking
```

---

# Retention Policy

Archived records remain available for:

```text
90 Days
```

After expiration:

```text
Automatic Purge
```

---

# Example Model

```json
{
  "id": "ol_01JX8N7W4T3M9K2R5A6B",
  "public_id": "k7m2x9qp",
  "account_id": "account_123",
  "block_id": "block_456",
  "block_type": "button",
  "destination_url": "https://wa.me/201234567890",
  "label_snapshot": "Book Consultation",
  "source_snapshot": {
    "type": "button",
    "title": "Book Consultation",
    "url": "https://wa.me/201234567890"
  },
  "status": "active",
  "archived_at": null,
  "created_at": "2026-01-01T12:00:00Z",
  "updated_at": "2026-01-01T12:00:00Z"
}
```

---

# Model Responsibilities

The Out Link Model is responsible for storing:

```text
Identity
Ownership
Destination
Clickable Snapshot
Lifecycle State
Audit Information
```

The model is not responsible for:

```text
Routing
Redirect Execution
Analytics Aggregation
Reporting
Presentation
```

These responsibilities belong to other modules.
