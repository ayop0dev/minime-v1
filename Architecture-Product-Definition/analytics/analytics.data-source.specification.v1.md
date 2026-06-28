# Analytics Data Source Specification V1

## Status

Approved

---

# Purpose

The Analytics Data Source Layer defines where analytics data originates and how it enters the Analytics System.

Analytics does not generate its own data.

Analytics consumes canonical events transported by Platform Events.

---

# Core Principle

```text
Analytics Consumes Platform Events
```

Analytics is a downstream system.

It depends on canonical events emitted by source domains and transported by Platform Events.

---

# Architecture Position

```text
Source Domains
↓
Platform Events
↓
Analytics System
↓
Aggregation
↓
Reports
```

---

# V1 Canonical Events

Analytics V1 consumes only:

```text
profile.viewed
out_link.followed
```

No additional event types exist.

---

# Source 1 — Public Profile Domain

Purpose:

```text
Profile View Measurement
```

The Public Profile domain emits:

```text
profile.viewed
```

through Platform Events after the profile has been successfully presented to the visitor.

Analytics consumes this event.

Analytics does not decide whether a view occurred.

---

# Profile View Event

A `profile.viewed` event is emitted by the Public Profile domain after the profile has been successfully presented to the visitor.

---

# View Counting Rule

Rule:

```text
Every profile.viewed Event
=
One Profile View
```

Repeated visits create additional events.

Page refreshes create additional events.

No visitor deduplication occurs.

No unique visitor calculation occurs.

This is an intentional V1 design decision.

---

# Profile View Source Data

Analytics receives from each `profile.viewed` event:

```text
account_id
timestamp
device_category
```

Only.

---

# Device Category

Supported values:

```text
desktop
mobile
tablet
```

Only.

This value is provided by the source domain.

The source domain determines the device category before the event is emitted.

Analytics receives the already-classified device category from the event.

Analytics treats device_category as immutable.

Analytics never recalculates it.

Analytics never reclassifies it.

No additional categories are supported.

---

# Source 2 — Out Link Domain

Purpose:

```text
Link Click Measurement
```

The Out Link domain emits:

```text
out_link.followed
```

through Platform Events after the Out Link has been successfully followed by a visitor.

Analytics consumes this event.

Analytics does not decide whether the Out Link was followed.

---

# Link Click Event

An `out_link.followed` event is emitted by the Out Link domain after the Out Link has been successfully followed by a visitor.

---

# Click Counting Rule

Rule:

```text
Every out_link.followed Event
=
One Link Click
```

No visitor deduplication occurs.

---

# Out Link Source Data

Analytics receives from each `out_link.followed` event:

```text
out_link_id
timestamp
```

Only.

---

# Event Ownership

Business domains own event creation.

Examples:

```text
Public Profile Domain emits profile.viewed
Out Link Domain emits out_link.followed
```

Platform Events transports them.

Analytics consumes them.

Analytics never creates business events.

---

# Analytics Responsibility

Analytics is responsible for:

```text
Reading Events from Platform Events
Aggregating Events
Calculating Metrics
Producing Reports
```

Analytics is not responsible for:

```text
Generating Source Events
Validating Source Events
Controlling Source Domains
Classifying Devices
```

---

# Append-Only Principle

Analytics events are append-only.

Existing analytics events are never modified.

Historical analytics are never rewritten.

If a correction is ever required, it is represented by a new event.

Existing events are never edited to reflect the correction.

---

# Event Integrity

Analytics assumes:

```text
Events Are Valid
```

when received from Platform Events.

Source domains are responsible for event correctness.

---

# Missing Event Principle

If an event does not exist:

```text
Analytics Cannot Invent It
```

Example:

```text
No out_link.followed Event
↓
No Click Count
```

---

# Event Storage Principle

Platform Storage owns persistence.

Analytics owns analytical meaning, schema, aggregation, and reporting.

Reports must be derived from stored events.

---

# V1 Unsupported Sources

Not supported:

```text
Visitor Tracking
Sessions
Cookies
Browser Data
Operating System Data
Geo Data
Referrer Data
Conversion Data
Revenue Data
Campaign Data
```

---

# Source Independence

Analytics must remain independent from source domain implementation details.

Analytics consumes:

```text
Canonical Events from Platform Events
```

not:

```text
Profiles
Blocks
Pages
Routes
UI Components
```

directly.

---

# Design Goal

The Data Source Layer should remain:

```text
Simple
Explicit
Event-Based
Extensible
```

Every analytics metric in V1 must be traceable back to:

```text
profile.viewed Events
or
out_link.followed Events
```

and nothing else.
