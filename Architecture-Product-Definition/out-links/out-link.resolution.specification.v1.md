# Out Link Resolution Specification V1

## Status

Approved

---

# Purpose

The Resolution Layer is responsible for converting a public Out Link identifier into a valid redirect destination.

Input:

```text
/out/k7m2x9qp
```

Output:

```text
https://youtube.com/@username
```

The Resolution Layer determines whether an Out Link can be used and what action should happen next.

---

# Position In System

```text
Visitor
↓
Routing
↓
Resolution
↓
Tracking
↓
Redirect
```

Resolution executes before:

```text
Tracking
Redirect Execution
Analytics Events
```

---

# Responsibilities

Resolution owns:

```text
Out Link Lookup
Status Verification
Destination Retrieval
Resolution Result Generation
```

Resolution does not own:

```text
Route Matching
Tracking
Analytics
Redirect Response
Error Page Rendering
```

---

# Input

Resolution receives:

```json
{
  "public_id": "k7m2x9qp"
}
```

From the Routing Layer.

---

# Lookup Process

Resolution searches:

```text
Out Link Model
```

Using:

```text
public_id
```

Example:

```text
k7m2x9qp
```

---

# Successful Lookup

If a matching record exists:

```text
Resolution Continues
```

and lifecycle validation begins.

---

# Missing Lookup

If no record exists:

```text
Resolution Fails
```

Result:

```text
LINK_NOT_FOUND
```

No tracking occurs.

No redirect occurs.

---

# Status Validation

After lookup:

```text
status
```

must be evaluated.

Allowed values:

```text
active
archived
```

Purged records do not exist.

---

# Active Status

If:

```text
status = active
```

Resolution succeeds.

Output:

```text
Destination URL
```

and control moves to:

```text
Tracking Layer
```

---

# Archived Status

If:

```text
status = archived
```

Resolution fails.

Result:

```text
LINK_ARCHIVED
```

No redirect occurs.

No click tracking occurs.

---

# Destination URL

Resolution retrieves:

```text
destination_url
```

from the Out Link Model.

Example:

```text
https://wa.me/201234567890
```

or

```text
https://youtube.com/@username
```

---

# Destination Validation

Resolution assumes:

```text
destination_url
```

was validated at creation time.

Resolution does not re-validate:

```text
Domain
Format
Platform Rules
```

during every request.

---

# Resolution Result

Successful resolution returns:

```json
{
  "result": "success",
  "public_id": "k7m2x9qp",
  "destination_url": "https://youtube.com/@username"
}
```

---

# Archived Result

Archived resolution returns:

```json
{
  "result": "archived",
  "public_id": "k7m2x9qp"
}
```

---

# Not Found Result

Missing resolution returns:

```json
{
  "result": "not_found",
  "public_id": "k7m2x9qp"
}
```

---

# Resolution Flow

Successful path:

```text
Receive Public ID
↓
Lookup Out Link
↓
Verify Active Status
↓
Retrieve Destination
↓
Return Success
```

---

# Archived Flow

```text
Receive Public ID
↓
Lookup Out Link
↓
Status = Archived
↓
Return Archived Result
```

---

# Missing Flow

```text
Receive Public ID
↓
Lookup Out Link
↓
No Record Found
↓
Return Not Found Result
```

---

# Resolution Output Contract

Resolution never performs:

```text
HTTP Redirect
Analytics Recording
Response Rendering
```

Resolution only returns:

```text
Resolution Result
```

to the next layer.

---

# Performance Goal

Resolution should require:

```text
Single Lookup
```

for successful requests.

The layer should remain:

```text
Fast
Deterministic
Stateless
```

---

# Security Rules

Resolution must never expose:

```text
Internal IDs
Account IDs
Block IDs
Ownership Data
```

Resolution only returns:

```text
Resolution Status
Destination URL
```

when appropriate.

---

# V1 Supported Outcomes

Resolution supports only:

```text
Success
Archived
Not Found
```

No additional decision trees exist in V1.

Not supported:

```text
Geo Routing
Conditional Routing
Device Routing
Weighted Routing
Campaign Routing
A/B Routing
```

---

# Design Goal

The Resolution Layer acts as the single source of truth for determining whether an Out Link can be used.

Its responsibility is limited to:

```text
Find Link
Validate State
Return Destination
```

and nothing more.
