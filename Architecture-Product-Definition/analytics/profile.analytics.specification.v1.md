# Profile Analytics Specification V1

## Status

Approved

---

# Purpose

Profile Analytics provides high-level performance metrics for a Minime profile.

Its purpose is to help Account owners understand overall profile activity.

---

# Core Principle

```text
Profile Analytics
=
Profile-Level Metrics
```

Profile Analytics measures profile performance.

It does not analyze individual visitors.

---

# Data Sources

Profile Analytics uses:

```text
Profile View Events
Link Click Events
```

Only.

Events are transported by Platform Events.

Analytics consumes canonical events from Platform Events only.

---

# Supported Metrics

Profile Analytics V1 supports:

```text
Total Views
Total Clicks
CTR
Device Breakdown
```

Only.

---

# Total Views

Definition:

```text
Total Number Of Profile Views
```

Calculated from:

```text
Profile View Events
```

Formula:

```text
Count(Profile View Events)
```

---

# Total Clicks

Definition:

```text
Total Number Of Link Clicks
```

Calculated from:

```text
Link Click Events
```

Formula:

```text
Count(Link Click Events)
```

---

# Click Through Rate (CTR)

Definition:

```text
Percentage Of Views That Produced A Click
```

Formula:

```text
Total Clicks / Total Views
```

Display:

```text
Percentage
```

Example:

```text
100 Views
20 Clicks

CTR = 20%
```

---

# Device Category Breakdown

Definition:

```text
Distribution Of Profile Views By Device Category
```

Supported device categories:

```text
Desktop
Mobile
Tablet
```

Only.

---

# Device Calculation

Calculated from:

```text
Profile View Events
```

Example:

```text
Desktop = 120
Mobile = 800
Tablet = 30
```

---

# Time Range Support

Profile Analytics may be filtered by:

```text
Today
Last 7 Days
Last 30 Days
All Time
```

Only.

---

# Repeated Views

Each `profile.viewed` event creates one additional Profile View record.

Repeated visits create additional events.

Page refreshes create additional events.

No visitor deduplication occurs.

No unique visitor calculation occurs.

This is an intentional V1 design decision.

---

# Profile Analytics Ownership

Profile Analytics belongs to:

```text
Account
```

and is calculated from all events associated with that account.

Account identity is provided by Platform Data.

Analytics references it.

Analytics does not own it.

---

# Relationship Model

```text
Account
 ├─ Profile View Events
 └─ Link Click Events
```

↓

```text
Profile Analytics
```

---

# V1 Non-Goals

Not supported:

```text
Unique Visitors
Sessions
Visitor Retention
Audience Analysis
Country Analytics
City Analytics
Browser Analytics
Operating System Analytics
Referrer Analytics
Conversion Analytics
Revenue Analytics
```

---

# Reporting Rules

Profile Analytics is:

```text
Aggregated
Read Only
Calculated
```

It is never manually modified.

The underlying analytics events are append-only.

Existing events are never modified.

Historical analytics are never rewritten.

---

# Design Goal

Profile Analytics should answer:

```text
How many times was my profile viewed?
How many clicks did my profile generate?
What is my CTR?
What device categories are being used?
```

using the smallest possible analytics surface area.
