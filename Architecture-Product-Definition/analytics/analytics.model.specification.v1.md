# Analytics Model Specification V1

## Status

Approved

---

# Purpose

The Analytics Model defines the analytics schema used by the Analytics System.

The model exists to organize analytics information derived from platform events and make it available for aggregation and reporting.

---

# Core Principle

```text
Analytics Stores Facts
Reports Calculate Metrics
```

The Analytics Model stores raw event information.

Derived metrics are calculated later.

---

# Architecture Position

```text
Platform Events
↓
Analytics Schema
↓
Platform Storage
↓
Aggregation
↓
Reports
```

---

# V1 Analytics Objects

Analytics V1 stores:

```text
Profile View Events
Link Click Events
```

Only.

---

# Event Categories

Supported categories:

```text
profile_view
link_click
```

No additional categories exist.

---

# Profile View Event Model

Represents:

```text
One profile.viewed Platform Event
```

Structure:

```json
{
  "event_id": "...",
  "event_type": "profile_view",
  "account_id": "...",
  "device_category": "mobile",
  "created_at": "..."
}
```

---

# Profile View Fields

## event_id

Purpose:

```text
Uniqueness
```

Rules:

```text
Required
Immutable
Unique
```

---

## event_type

Value:

```text
profile_view
```

---

## account_id

Represents:

```text
Account Owner
```

Identity provided by Platform Data.

Analytics references this identity.

Analytics does not own it.

Rules:

```text
Required
Immutable
```

---

## device_category

Allowed values:

```text
desktop
mobile
tablet
```

This value is received from the `profile.viewed` event.

The source domain determines the category before the event reaches Analytics.

Analytics treats device_category as immutable.

Analytics never recalculates it.

Analytics never reclassifies it.

Aggregation only groups existing values.

Rules:

```text
Required
Immutable
```

---

## created_at

Purpose:

```text
Event Timestamp
```

Rules:

```text
UTC
Immutable
Required
```

---

# Link Click Event Model

Represents:

```text
One out-link.clicked Platform Event
```

Structure:

```json
{
  "event_id": "...",
  "event_type": "link_click",
  "out_link_id": "...",
  "created_at": "..."
}
```

---

# Link Click Fields

## event_id

Purpose:

```text
Uniqueness
```

Rules:

```text
Required
Immutable
Unique
```

---

## event_type

Value:

```text
link_click
```

---

## out_link_id

Represents:

```text
Clicked Out Link
```

Identity provided by Platform Data.

Analytics references this identity.

Analytics does not own it.

Rules:

```text
Required
Immutable
```

---

## created_at

Purpose:

```text
Event Timestamp
```

Rules:

```text
UTC
Immutable
Required
```

---

# Event Immutability

Analytics events are append-only records.

After creation:

```text
Events Cannot Be Modified
```

Existing analytics events are never modified.

Historical analytics are never rewritten.

If a correction is ever required, it is represented by a new event.

Existing events are never edited to reflect the correction.

Allowed:

```text
Create
Read
Aggregate
```

Not Allowed:

```text
Update
Overwrite
Rewrite
Delete Individual Events
```

---

# Storage Principle

Platform Storage owns:

```text
Persistence
Retention
Physical Storage
```

Analytics owns:

```text
Analytics Records
Aggregation
Reporting
Metric Calculation
```

Platform Storage owns persistence.

Analytics owns analytical meaning, schema, aggregation, and reporting.

---

# Relationship Model

```text
Account (identity from Platform Data)
 └─ Profile View Events

Out Link (identity from Platform Data)
 └─ Link Click Events
```

---

# Aggregation Inputs

The model provides inputs for:

```text
Profile Views
Total Clicks
CTR
Device Category Breakdown
Top Links
```

---

# Retention

Retention duration is defined by Platform Storage.

Analytics follows the retention rules defined for analytics events.

---

# V1 Non-Goals

The model does not store:

```text
Visitor Identity
Sessions
Cookies
Browser Data
Operating System Data
Geo Data
Referrer Data
Campaign Data
Revenue Data
```

---

# Design Goal

The Analytics Model should remain:

```text
Simple
Append-Only
Event-Based
Scalable
```

Every metric in V1 must be derivable from:

```text
Profile View Events
Link Click Events
```

and nothing else.
