# Public Profile Request Lifecycle V1

## Status

Approved

---

# Purpose

This document defines the complete lifecycle of a public profile request in Minime V1.

It describes the sequence of systems involved from the moment a visitor requests a profile until a final response is returned.

---

# Lifecycle Philosophy

The request lifecycle answers:

```text
What happens when someone visits a profile?
```

It does not define:

```text
Profile Content

Rendering Rules

Theme Rules

Layout Rules
```

Those responsibilities belong to other systems.

---

# Lifecycle Overview

```text
Visitor Request
↓
Routing
↓
Profile Resolution
↓
Access Validation
↓
Rendering Output Retrieval
↓
Response Delivery
```

---

# Step 1 — Visitor Request

Example:

```text
https://minime.ae/ahmed
```

A visitor requests a public profile URL.

---

# Step 2 — Routing

The Routing System extracts:

```json
{
  "username": "ahmed"
}
```

Routing does not load profiles.

Routing only identifies the requested username.

---

# Step 3 — Profile Resolution

The Resolution System determines:

```text
Which Account owns this username?
```

Example output:

```json
{
  "account_id": "123",
  "username": "ahmed"
}
```

---

# Step 4 — Access Validation

The Access Policy determines:

```text
Can this request be served?
```

Possible results:

```text
Allow

Deny
```

---

# Step 5 — Rendering Output Retrieval

If access is allowed:

```text
Load Rendered Output
```

The system retrieves the prepared profile output from the Rendering layer.

The Public Profile layer does not:

```text
Render Blocks

Apply Themes

Apply Layouts
```

Those responsibilities have already been completed upstream.

---

# Step 6 — Response Delivery

Successful result:

```text
Return Public Profile
```

Example:

```text
HTTP 200
```

with profile content.

---

# Successful Lifecycle

```text
Request
↓
Routing
↓
Resolution
↓
Access Policy
↓
Rendering Output Retrieval
↓
200 OK
```

---

# Failed Lifecycle — Profile Not Found

```text
Request
↓
Routing
↓
Resolution Failed
↓
404
```

---

# Failed Lifecycle — Access Denied

```text
Request
↓
Routing
↓
Resolution
↓
Access Policy Denied
↓
404
```

---

# Public Information Policy

Visitors must never learn whether:

```text
The Account Never Existed

The Account Was Deleted

The Account Is Suspended
```

All denied states produce the same public outcome.

---

# Error Handling

Unexpected failures:

```text
Internal System Error
```

Result:

```text
500 Internal Server Error
```

Detailed handling is defined in:

```text
public-profile.error.states.v1.md
```

---

# Relationship With Cache

Caching may occur before presentation retrieval.

Example:

```text
Request
↓
Cache Hit
↓
Return Response
```

or

```text
Request
↓
Cache Miss
↓
Continue Lifecycle
```

Caching never changes lifecycle decisions.

---

# Relationship With Rendering

The lifecycle consumes Rendered Profile Output.

The lifecycle never:

```text
Creates Themes

Creates Layouts

Creates Render Objects

Transforms Content
```

---

# V1 Scope

Supported:

```text
Public Profile Requests

Username Routing

Profile Resolution

Access Validation

Rendering Output Retrieval

Response Delivery
```

Not Supported:

```text
Subpages

Custom Domains

Authenticated Public Pages

Regional Delivery Rules
```

---

# Architectural Rule

Each stage owns exactly one responsibility:

```text
Routing
→ Username

Resolution
→ Account

Access Policy
→ Access Decision

Rendering
→ Output

Delivery
→ Response
```

Responsibilities must never overlap.

---

# Core Principle

A public profile request is a pipeline.

Each system performs one task.

Then passes control to the next system.
