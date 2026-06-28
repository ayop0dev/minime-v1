# Out Link Tracking Policy V1

## Status

Approved

---

# Purpose

The Tracking Layer is responsible for observing outbound link activity and recording click events.

Tracking exists to provide analytics data.

Tracking does not influence:

```text
Resolution
Routing
Redirect Decisions
Link Lifecycle
```

---

# Core Principle

```text
Tracking Is Observational
```

Tracking records what happened.

Tracking never decides what should happen.

---

# System Position

```text
Routing
↓
Resolution
↓
Redirect Response
↓
Tracking
↓
Click Event Recorded
```

---

# Tracking Trigger

A click is recorded only when:

```text
Redirect Response Sent
```

Examples:

```text
302 Sent
```

Result:

```text
Click Event Created
```

---

# Not Tracked

No click event is created when:

```text
Resolution Failed
Link Not Found
Link Archived
Redirect Not Sent
```

---

# V1 Event Model

V1 tracks:

```text
One Event
=
One Redirect
```

No aggregation occurs at tracking time.

---

# Duplicate Clicks

Every successful redirect is tracked.

Example:

```text
Same Visitor
↓
Clicks 10 Times
```

Result:

```text
10 Click Events
```

No deduplication occurs in V1.

---

# Why No Deduplication

Reason:

```text
Simple
Predictable
Auditable
```

Analytics may aggregate later.

Tracking records raw activity.

---

# Required Tracking Data

Every click event must include:

```text
Out Link Identifier
Timestamp
```

---

# Optional Request Metadata

V1 may collect:

```text
Referrer
User Agent
IP Address
```

for operational purposes.

These values are not required for analytics correctness.

---

# IP Address Policy

V1 may temporarily process IP addresses.

Purpose:

```text
Security
Abuse Detection
Rate Limiting
```

IP addresses must not become part of public analytics.

---

# Public Analytics Rule

Public analytics must never expose:

```text
IP Address
Exact Location
Personal Information
```

---

# Country Tracking

V1 does not perform:

```text
Geo Lookup
Country Resolution
Region Analytics
```

Not supported.

---

# City Tracking

V1 does not track:

```text
City
Region
Coordinates
```

Not supported.

---

# Device Tracking

V1 does not classify:

```text
Desktop
Mobile
Tablet
```

Not supported.

Device classification is not performed during click tracking in V1.

Device type is collected at the profile view analytics level, not at the Out Link click tracking level.

---

# Browser Tracking

V1 does not classify:

```text
Chrome
Safari
Firefox
Edge
```

Not supported.

---

# Operating System Tracking

V1 does not classify:

```text
iOS
Android
Windows
macOS
Linux
```

Not supported.

---

# Session Tracking

V1 does not maintain:

```text
Visitor Sessions
Session Duration
Session Identity
```

Not supported.

---

# Visitor Tracking

V1 does not identify:

```text
Visitors
Users
Devices
Individuals
```

Not supported.

---

# Cookie Policy

V1 does not require:

```text
Cookies
Visitor IDs
Fingerprinting
```

Not supported.

---

# Bot Detection

V1 does not perform:

```text
Bot Detection
Traffic Scoring
Fraud Analysis
```

Not supported.

---

# Tracking Reliability

Tracking failures must never block redirects.

Example:

```text
Tracking Error
```

Result:

```text
Redirect Continues
```

---

# Failure Principle

Priority order:

```text
Visitor Experience
↓
Redirect Success
↓
Analytics
```

Analytics must never prevent:

```text
Successful Redirect
```

---

# Retention

Tracking records follow the lifecycle of the Out Link.

When an Out Link becomes:

```text
Archived
```

historical tracking remains available.

When an Out Link becomes:

```text
Purged
```

tracking records may also be removed.

---

# Data Ownership

Tracking data belongs to:

```text
Out Link System
```

Not:

```text
Profile Content
Blocks
Presentation
```

---

# Analytics Relationship

Tracking generates:

```text
Raw Events
```

Analytics consumes:

```text
Raw Events
```

Tracking never generates reports.

Tracking never calculates metrics.

---

# V1 Supported Metrics Source

Tracking provides data for:

```text
Total Clicks
Link Performance
Future Analytics
```

through raw event generation.

---

# Design Goal

The Tracking Layer should remain:

```text
Simple
Reliable
Append-Only
Independent
```

Its only responsibility is:

```text
Observe Redirects
Record Events
Store Facts
```

and nothing more.
