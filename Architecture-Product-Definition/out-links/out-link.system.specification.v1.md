# Out Link System Specification V1

## Status

Approved

---

# Purpose

The Out Link System is responsible for handling all outbound link traffic originating from public Minime profiles.

Instead of sending visitors directly to destination URLs, all clicks pass through the Out Link System.

Example:

```text
Visitor
↓
/out/k7m2x9qp
↓
Destination URL
```

This allows Minime to:

```text
Track Link Activity
Measure Performance
Record Analytics Events
Control Redirect Lifecycle
Resolve Link Destinations
```

---

# Scope

The Out Link System owns:

```text
Out Link Creation
Out Link Resolution
Outbound Redirects
Link Tracking
Link Lifecycle
Analytics Event Generation
```

The Out Link System does not own:

```text
Profile Content
Blocks
Rendering
Themes
Presentation
Profile Analytics Reporting
```

---

# System Position

```text
Profile Content
↓
Blocks
↓
Public Profile
↓
Visitor Click
↓
Out Link System
↓
Destination URL
```

---

# Core Principle

Out Links exist to represent clickable actions.

An Out Link does not represent:

```text
A Profile
A Block
A Destination URL
```

An Out Link represents:

```text
A Clickable State
```

---

# Clickable State Definition

A Clickable State is the complete clickable configuration presented to the visitor.

Example:

```text
Button Text:
Book Consultation

Destination:
https://wa.me/...
```

This combination forms a single Clickable State.

---

# Identity Model

```text
Out Link Identity
=
Clickable State Identity
```

Changing the Clickable State creates a new Out Link.

Changing unrelated visual settings does not.

---

# Ownership Model

Out Links belong to an Account.

Structure:

```text
Account
 ├─ Blocks
 └─ Out Links
```

Each Out Link may reference the Block that generated it.

Structure:

```text
Account
 └─ Out Link
      └─ Attached Block
```

Out Links are not owned by Blocks.

---

# Out Link Creation

A new Out Link is created whenever a new Clickable State is created.

Examples:

```text
New Button
New Social Link
New Clickable Block
```

Result:

```text
New Out Link
```

---

# Clickable State Changes

The following changes generate a new Out Link:

```text
Destination URL Change
Button Text Change
Social Platform Change
Social URL Change
Block Type Change
```

Reason:

```text
New Clickable State
```

---

# Visual Changes

The following changes do not generate a new Out Link:

```text
Theme Change
Background Change
Block Position Change
Block Order Change
Spacing Change
Presentation Change
```

Reason:

```text
Clickable State Unchanged
```

---

# Identifier Format

Public format:

```text
/out/k7m2x9qp
```

Rules:

```text
8 Characters
Lowercase
Letters and Numbers
URL Safe
Non-Sequential
Non-Guessable
```

Allowed Characters:

```text
a-z
0-9
```

Example:

```text
/out/8f7k29xq
/out/m2t7xk9a
/out/p7j4s1cw
```

---

# Redirect Flow

Standard flow:

```text
Visitor Click
↓
Out Link Resolution
↓
Tracking
↓
Redirect
↓
Destination URL
```

---

# Analytics Relationship

Every Out Link may generate analytics events.

Examples:

```text
Link Click
Redirect Success
Redirect Failure
```

Analytics event definitions are specified separately.

---

# Lifecycle States

Every Out Link must exist in exactly one state.

States:

```text
Created
Active
Archived
Purged
```

---

# Created

The Out Link has been generated but is not yet active.

State duration is typically temporary.

Flow:

```text
Created
↓
Active
```

---

# Active

The Out Link is publicly accessible.

Capabilities:

```text
Resolves Destination
Redirects Visitors
Generates Analytics Events
Participates In Tracking
```

---

# Archived

The Out Link is no longer active.

Capabilities:

```text
No Redirects
No New Tracking
Historical Data Retained
```

Retention:

```text
90 Days
```

Purpose:

```text
Historical Analysis
Lifecycle Integrity
Data Consistency
```

---

# Purged

The Out Link has been permanently removed.

Capabilities:

```text
No Resolution
No Redirect
No Tracking
No Historical Access
```

Purging occurs automatically after archive retention expires.

---

# Lifecycle Flow

```text
Created
↓
Active
↓
Archived (90 Days)
↓
Purged
```

---

# V1 Supported Sources

Out Links may be generated from:

```text
Button Blocks
Social Blocks
Future Clickable Blocks
```

The system is source-agnostic.

Any future clickable element may create Out Links without changing the system architecture.

---

# V1 Non-Goals

The Out Link System does not provide:

```text
Affiliate Management
Revenue Attribution
A/B Testing
Smart Routing
Geo Routing
Device Routing
Conditional Redirects
Campaign Logic
```

These may be introduced in future versions.

---

# Design Goal

The Out Link System should remain:

```text
Simple
Predictable
Trackable
Auditable
Extensible
```

while serving as the single source of truth for all outbound profile traffic inside Minime.
