# Analytics System Specification V1

## Status

Approved

---

# Purpose

The Analytics System is responsible for measuring basic public profile activity inside Minime.

It provides simple performance insights for Account owners without introducing complex visitor tracking.

---

# Core Principle

```text
Analytics V1
=
Simple Activity Measurement
```

Analytics V1 measures what happened.

It does not identify who did it.

---

# Scope

Analytics V1 supports:

```text
Profile Views
Total Link Clicks
Clicks Per Link
CTR
Device Category Breakdown
```

Only.

---

# Not Supported In V1

Analytics V1 does not support:

```text
Unique Visitors
Visitor Identity
Sessions
Cookies
Fingerprinting
Geo Analytics
Country Analytics
City Analytics
Browser Analytics
Operating System Analytics
Device Model Analytics
Referrer Analytics
Conversion Tracking
Funnels
Revenue Attribution
```

---

# System Position

```text
Public profile surface
↓
Platform Events
↓
Analytics
```

```text
Out Link Domain
↓
Platform Events
↓
Analytics
```

Analytics consumes canonical events from Platform Events.

Analytics does not depend on implementation details of source domains.

Analytics does not control source domains.

---

# Data Sources

Analytics V1 consumes events from Platform Events only.

Platform Events carries:

```text
profile.viewed
out-link.clicked
```

Source domains emit these events.

Platform Events transports them.

Analytics consumes them.

---

# Profile Views

A Profile View is represented by a canonical `profile.viewed` event.

That event is emitted by the public profile surface (produced by Rendering) after the profile has been successfully presented to the visitor.

Analytics does not decide whether a view occurred.

Analytics only records the emitted `profile.viewed` event.

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

# Link Clicks

A Link Click is represented by a canonical `out-link.clicked` event.

That event is emitted by the Out Link domain after the Out Link has been successfully followed by a visitor.

Analytics does not decide whether the Out Link was followed.

Analytics only records the emitted `out-link.clicked` event.

Rule:

```text
Every out-link.clicked Event
=
One Link Click
```

---

# CTR

CTR means:

```text
Click Through Rate
```

In V1, CTR is calculated as:

```text
Total Link Clicks / Profile Views
```

CTR is a reporting metric only.

It is not stored as a raw event.

---

# Device Category Breakdown

Analytics V1 recognizes:

```text
desktop
mobile
tablet
```

Only.

Analytics never classifies devices.

The source domain determines the device category before the event is emitted.

Analytics receives the already-classified device category from the `profile.viewed` event.

Analytics treats device_category as immutable.

Analytics never recalculates it.

Analytics never reclassifies it.

Aggregation only groups existing values.

The system does not expose:

```text
Browser
Operating System
Device Model
Exact Screen Size
```

---

# Event-Based Architecture

Analytics V1 is event-based.

Analytics events are append-only.

Raw events are recorded first.

Existing events are never modified.

Historical analytics are never rewritten.

Reports are calculated from events.

```text
Raw Events
↓
Aggregation
↓
Reports
```

---

# Analytics Ownership

Platform Storage owns:

```text
Persistence
Retention
Physical Storage
```

Platform Data owns:

```text
Account Identity
Out Link Identity
```

The Analytics System owns:

```text
Analytics Schema
Metric Aggregation
Analytics Reporting
Privacy Rules
```

The Analytics System does not own:

```text
Profile Content
Blocks
Out Link Resolution
Redirect Execution
Public Profile Rendering
Presentation Themes
```

---

# Privacy Principle

Analytics V1 must remain privacy-light.

It must not identify visitors.

It must not build visitor profiles.

It must not require cookies.

It must not use fingerprinting.

---

# V1 Dashboard Goal

The V1 dashboard should answer:

```text
How many times was my profile viewed?
How many times were my links clicked?
Which links performed best?
What is my click-through rate?
What device categories are visitors using?
```

Only.

---

# Design Goal

The Analytics System should remain:

```text
Simple
Useful
Privacy-Light
Event-Based
Extensible
```

It should provide enough insight for V1 without turning Minime into a complex tracking platform.
