# Analytics Aggregation Policy V1

## Status

Approved

---

# Purpose

The Aggregation Layer is responsible for transforming raw analytics events into metrics that can be displayed in reports.

Aggregation does not create events.

Aggregation calculates values from existing events.

---

# Core Principle

```text
Events Are Facts
Metrics Are Calculations
```

Events are stored.

Metrics are derived.

---

# System Position

```text
Raw Events
↓
Aggregation
↓
Metrics
↓
Reports
```

---

# Input Sources

Aggregation consumes:

```text
Profile View Events
Link Click Events
```

Only.

No additional inputs exist.

---

# Output Metrics

Aggregation produces:

```text
Total Views
Total Clicks
CTR
Device Breakdown
Link Ranking
```

Only.

---

# Total Views Aggregation

Definition:

```text
Count All Profile View Events
```

Formula:

```text
Total Views
=
Count(Profile View Events)
```

---

# Total Clicks Aggregation

Definition:

```text
Count All Link Click Events
```

Formula:

```text
Total Clicks
=
Count(Link Click Events)
```

---

# CTR Aggregation

Definition:

```text
Click Through Rate
```

Formula:

```text
CTR
=
Total Clicks
/
Total Views
```

Display:

```text
Percentage
```

Example:

```text
100 Views
20 Clicks
↓
20%
```

---

# Device Category Breakdown Aggregation

Definition:

```text
Group Profile View Events
By Device Category
```

Device Category Breakdown is derived from Profile View Events only.

Link Click Events do not carry device information and are not used to calculate Device Category Breakdown.

Supported device categories:

```text
Desktop
Mobile
Tablet
```

Only.

Device category values are determined by the source domain before Analytics receives the event.

Aggregation only groups existing values.

Aggregation never recalculates or reclassifies device categories.

---

# Device Calculation

Example:

```text
Desktop = 120
Mobile = 840
Tablet = 40
```

Calculated from:

```text
Profile View Events
```

---

# Link Ranking Aggregation

Definition:

```text
Order Out Links
By Click Count
```

Highest click count appears first.

---

# Link Ranking Example

```text
YouTube = 120 Clicks
WhatsApp = 80 Clicks
LinkedIn = 25 Clicks
```

Result:

```text
1. YouTube
2. WhatsApp
3. LinkedIn
```

---

# Aggregation Scope

Aggregation may occur for:

```text
Entire Profile
Single Out Link
Specific Time Range
```

---

# Supported Time Ranges

V1 supports:

```text
Today
Last 7 Days
Last 30 Days
All Time
```

Only.

---

# Calculation Principle

Metrics are never stored as permanent values.

Metrics are always derived from events.

Example:

```text
Total Views
```

is calculated.

Not stored.

---

# Data Integrity

Analytics events are append-only.

Aggregation must never modify:

```text
Profile View Events
Link Click Events
```

Existing events are never modified.

Historical analytics are never rewritten.

Events remain immutable.

---

# Missing Data Principle

If no events exist:

```text
Metric Value = 0
```

No estimation occurs.

No prediction occurs.

---

# Historical Integrity

Aggregation must respect:

```text
Out Link Identity
=
Clickable State Identity
```

Analytics never merge multiple Out Links.

Example:

```text
Out Link A
100 Clicks

Out Link B
50 Clicks
```

Result:

```text
A = 100
B = 50
```

Never:

```text
150 Combined Clicks
```

---

# Aggregation Ownership

Aggregation owns:

```text
Metric Calculation
Grouping
Counting
Ranking
```

Aggregation does not own:

```text
Event Creation
Reporting
Presentation
```

---

# V1 Non-Goals

Not supported:

```text
Predictions
Trend Forecasting
Machine Learning
Anomaly Detection
Visitor Segmentation
Behavior Analysis
Attribution Modeling
```

---

# Design Goal

The Aggregation Layer should remain:

```text
Deterministic
Transparent
Simple
```

Every metric must be traceable back to:

```text
Profile View Events
or
Link Click Events
```

with no hidden calculations.
