# Out Link Analytics Event Specification V1

## Status

Approved

---

# Purpose

Analytics Events represent recorded outbound link activity.

An Analytics Event is created when a redirect is successfully sent to a visitor.

The event becomes the source of truth for click measurement.

---

# Core Principle

```text
One Redirect
=
One Analytics Event
```

Every successful redirect creates exactly one event.

---

# Event Creation Trigger

Event creation occurs when:

```text
Redirect Response Sent
```

Examples:

```text
302 Redirect Sent
307 Redirect Sent
```

Result:

```text
Analytics Event Created
```

---

# Event Creation Rules

Create Event:

```text
Active Out Link
Redirect Sent Successfully
```

Do Not Create Event:

```text
Link Not Found
Archived Link
Resolution Failure
Redirect Failure
```

---

# Event Purpose

Events exist to answer:

```text
How many clicks happened?
```

V1 does not answer:

```text
Who clicked?
Where did they click from?
What device was used?
What browser was used?
```

---

# Analytics Event Model

Each event contains:

```text
event_id
out_link_id
created_at
```

Only.

---

# Event Identifier

Field:

```text
event_id
```

Purpose:

```text
Uniqueness
Auditing
Storage
```

Requirements:

```text
Unique
System Generated
Immutable
```

---

# Out Link Reference

Field:

```text
out_link_id
```

Purpose:

```text
Event Attribution
Click Counting
Analytics Aggregation
```

Rules:

```text
Required
Immutable
```

---

# Timestamp

Field:

```text
created_at
```

Purpose:

```text
Event Timing
Aggregation
Auditing
```

Rules:

```text
Required
Immutable
UTC
```

---

# Example Event

```json
{
  "event_id": "evt_01JX9R6T4M8N2Q7W5K",
  "out_link_id": "ol_01JX8N7W4T3M9K2R5A6B",
  "created_at": "2026-01-01T12:00:00Z"
}
```

---

# Event Immutability

After creation:

```text
Events Cannot Be Modified
```

Events are append-only records.

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
Mutate
```

---

# Aggregation

Analytics systems may calculate:

```text
Total Clicks
Clicks Per Link
Clicks Per Profile
```

using event counts.

Aggregation is not part of event storage.

---

# Ownership

Events belong to:

```text
Out Link System
```

and are associated with:

```text
Out Links
```

Events do not belong directly to:

```text
Profiles
Blocks
Presentation
```

---

# Lifecycle Relationship

Events inherit the lifecycle of their Out Link.

Active:

```text
Events Available
```

Archived:

```text
Events Available
```

Purged:

```text
Events May Be Removed
```

according to retention policy.

---

# V1 Supported Analytics

Events support:

```text
Click Counting
Basic Link Performance
Basic Profile Performance
```

Only.

---

# V1 Non-Goals

Not supported:

```text
Visitor Analytics
Session Analytics
Device Analytics
Country Analytics
Browser Analytics
Conversion Analytics
Attribution Analytics
Revenue Analytics
Behavior Analytics
```

---

# Design Goal

The Analytics Event System should remain:

```text
Minimal
Immutable
Auditable
Scalable
```

The only responsibility of an Analytics Event is:

```text
Record That A Click Happened
```

and nothing more.
