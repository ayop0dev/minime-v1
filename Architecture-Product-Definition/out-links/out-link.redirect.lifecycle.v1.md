# Out Link Redirect Lifecycle Specification V1

## Status

Approved

---

# Purpose

The Redirect Lifecycle defines the complete execution flow of an outbound link request.

It describes what happens from the moment a visitor opens an Out Link until the request is completed.

---

# Scope

The lifecycle coordinates:

```text
Routing
Resolution
Tracking
Redirect Response
```

The lifecycle does not define:

```text
Analytics Reporting
Profile Rendering
Rendering
Presentation Logic
```

---

# Lifecycle Goal

The objective is:

```text
Receive Request
Validate Link
Redirect Visitor
Record Click
```

using the simplest possible flow.

---

# Entry Point

A lifecycle begins when a visitor requests:

```text
/out/{public_id}
```

Example:

```text
/out/k7m2x9qp
```

---

# Step 1 — Routing

The Routing Layer receives:

```text
/out/k7m2x9qp
```

Routing:

```text
Matches Route
Extracts public_id
Validates Format
```

Output:

```text
public_id
```

---

# Step 2 — Resolution

The Resolution Layer receives:

```text
public_id
```

Resolution:

```text
Looks Up Out Link
Validates Status
Retrieves Destination URL
```

Possible outcomes:

```text
Success
Archived
Not Found
```

---

# Success Path

If Resolution succeeds:

```text
Destination URL Available
```

Flow continues.

Example:

```text
https://youtube.com/@username
```

---

# Archived Path

If Resolution returns:

```text
Archived
```

Lifecycle stops.

No redirect occurs.

No click event is created.

Control moves to:

```text
Error Handling
```

---

# Not Found Path

If Resolution returns:

```text
Not Found
```

Lifecycle stops.

No redirect occurs.

No click event is created.

Control moves to:

```text
Error Handling
```

---

# Step 3 — Redirect Preparation

When Resolution succeeds:

```text
Destination URL Available
```

The system prepares:

```text
Redirect Response
```

---

# Step 4 — Redirect Response

The system sends:

```text
HTTP Redirect
```

to the visitor.

V1 uses:

```text
302 Temporary Redirect
```

for all outbound links.

---

# Redirect Principle

Out Links are not permanent URLs.

They represent:

```text
Dynamic Clickable States
```

Therefore:

```text
302
```

is preferred over:

```text
301
```

---

# Step 5 — Click Event Creation

After the redirect response is successfully sent:

```text
Analytics Event Created
```

Event creation uses:

```text
Out Link Identifier
Timestamp
```

Only.

---

# Tracking Principle

The click event records:

```text
A Redirect Happened
```

Nothing more.

---

# Successful Lifecycle

Complete successful flow:

```text
Visitor
↓
Request /out/{public_id}
↓
Routing
↓
Resolution
↓
Destination Found
↓
302 Redirect Sent
↓
Click Event Created
↓
Lifecycle Complete
```

---

# Archived Lifecycle

```text
Visitor
↓
Request /out/{public_id}
↓
Routing
↓
Resolution
↓
Archived Link
↓
Lifecycle Stops
↓
Error Handling
```

---

# Not Found Lifecycle

```text
Visitor
↓
Request /out/{public_id}
↓
Routing
↓
Resolution
↓
Link Not Found
↓
Lifecycle Stops
↓
Error Handling
```

---

# Failure Handling

If any step fails before redirect:

```text
No Redirect
No Click Event
```

Lifecycle ends.

---

# Tracking Failure

If click event creation fails:

```text
Redirect Must Continue
```

Visitor experience has priority.

---

# Priority Order

```text
Successful Redirect
↓
Analytics Recording
```

Never:

```text
Analytics
↓
Redirect
```

---

# State Awareness

The lifecycle respects:

```text
Active
Archived
```

states.

Purged records are treated as:

```text
Not Found
```

because no record exists.

---

# V1 Supported Outcomes

Successful outcomes:

```text
Redirect Sent
Click Recorded
```

Failure outcomes:

```text
Archived
Not Found
```

Only.

---

# V1 Non-Goals

Not supported:

```text
Conditional Redirects
Geo Redirects
Device Redirects
Traffic Distribution
Link Rotation
Campaign Redirects
A/B Redirects
```

---

# Design Goal

The Redirect Lifecycle should remain:

```text
Linear
Predictable
Fast
Auditable
```

The system should always follow:

```text
Route
↓
Resolve
↓
Redirect
↓
Record
```

with no additional decision layers.
