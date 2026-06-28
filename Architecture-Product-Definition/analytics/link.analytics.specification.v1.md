# Link Analytics Specification V1

## Status

Approved

---

# Purpose

Link Analytics provides performance metrics for individual Out Links.

Its purpose is to help Account owners understand which links receive the most engagement.

---

# Core Principle

```text
Link Analytics
=
Out Link Analytics
```

Analytics is calculated per Out Link.

Analytics is not calculated per:

```text
Button
Block
Component
```

because these elements may change over time.

---

# Analytics Unit

The primary analytics unit is:

```text
Out Link
```

Each Out Link represents:

```text
One Clickable State
```

---

# Data Source

Link Analytics uses:

```text
Link Click Events
```

Only.

Events are transported by Platform Events.

Analytics consumes canonical `out_link.followed` events from Platform Events only.

---

# Supported Metrics

Link Analytics V1 supports:

```text
Total Clicks
Link Ranking
```

Only.

---

# Total Clicks

Definition:

```text
Total Number Of Click Events
Associated With An Out Link
```

Formula:

```text
Count(Link Click Events)
```

Example:

```text
Out Link A = 120 Clicks
Out Link B = 75 Clicks
Out Link C = 12 Clicks
```

---

# Link Ranking

Definition:

```text
Ordering Links By Click Count
```

Highest click count appears first.

Example:

```text
1. YouTube → 120 Clicks
2. WhatsApp → 75 Clicks
3. LinkedIn → 12 Clicks
```

---

# Analytics Ownership

Link Analytics belongs to:

```text
Out Links
```

not:

```text
Blocks
Buttons
Social Components
```

Out Link identity is provided by Platform Data.

Analytics references it.

Analytics does not own it.

---

# Historical Integrity

Because:

```text
Out Link Identity
=
Clickable State Identity
```

every new Clickable State creates:

```text
New Out Link
```

Therefore:

```text
Analytics Never Moves
```

between Out Links.

---

# Example

Initial state:

```text
Visit Website
↓
Out Link A
↓
100 Clicks
```

Later:

```text
Watch On YouTube
↓
Out Link B
↓
0 Clicks
```

Result:

```text
Out Link A = 100 Clicks
Out Link B = 0 Clicks
```

Analytics remain attached to the original Out Link.

---

# Archived Out Links

Archived Out Links retain historical analytics while archived.

Example:

```text
Out Link Archived
↓
Click History Preserved
```

until purge occurs.

---

# Purged Out Links

When an Out Link is purged:

```text
Associated Analytics
May Also Be Removed
```

according to retention policies.

---

# Link Display Information

Analytics may display:

```text
Label Snapshot
Destination URL
Click Count
```

for reporting purposes.

These values are derived from the Out Link record.

---

# Time Range Support

Link Analytics may be filtered by:

```text
Today
Last 7 Days
Last 30 Days
All Time
```

Only.

---

# Relationship Model

```text
Out Link
 └─ Link Click Events
```

↓

```text
Link Analytics
```

---

# V1 Non-Goals

Not supported:

```text
CTR Per Link
Conversion Rate
Visitor Analysis
Country Analysis
Device Analysis
Browser Analysis
Revenue Analysis
Attribution Analysis
```

---

# Reporting Rules

Link Analytics is:

```text
Aggregated
Read Only
Calculated
```

Analytics values are never manually modified.

The underlying analytics events are append-only.

Existing events are never modified.

Historical analytics are never rewritten.

---

# Design Goal

Link Analytics should answer:

```text
Which links perform best?
How many clicks did each link receive?
How do links compare against each other?
```

while remaining fully aligned with:

```text
Out Link Identity
=
Clickable State Identity
```

and nothing else.
