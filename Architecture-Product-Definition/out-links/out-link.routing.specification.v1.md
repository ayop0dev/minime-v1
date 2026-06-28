# Out Link Routing Specification V1

## Status

Approved

---

# Purpose

The Routing Layer is responsible for receiving public Out Link requests and forwarding them to the Resolution Layer.

Routing does not perform:

```text
Destination Resolution
Redirect Logic
Tracking
Analytics
Lifecycle Decisions
```

Routing only identifies the request and passes control to the next system layer.

---

# Scope

Routing owns:

```text
Public Route Matching
Identifier Extraction
Request Validation
Resolution Dispatch
```

Routing does not own:

```text
Out Link Lookup
Destination Selection
Redirect Execution
Analytics
Error Handling Logic
```

---

# Route Structure

Public format:

```text
/out/{public_id}
```

Example:

```text
/ out/k7m2x9qp
/out/m2t7xk9a
/out/p7j4s1cw
```

---

# Route Ownership

The route belongs to:

```text
Out Link System
```

Not:

```text
Profile System
Public Profile System
Block System
```

---

# Identifier Source

The identifier is:

```text
public_id
```

From the Out Link Model.

Example:

```text
public_id = k7m2x9qp
```

Produces:

```text
/out/k7m2x9qp
```

---

# Route Matching

A request matches when:

```text
Path Begins With /out/
```

and:

```text
A Public Identifier Exists
```

Example:

```text
/out/k7m2x9qp
```

Valid.

---

# Invalid Routes

Examples:

```text
/out
/out/
/out//
```

Invalid.

---

# Identifier Validation

Before resolution begins:

```text
public_id
```

must satisfy:

```text
Length = 8
Characters = a-z, 0-9
Lowercase Only
```

---

# Valid Examples

```text
k7m2x9qp
8f7k29xq
m2t7xk9a
```

---

# Invalid Examples

```text
ABC12345
abc-1234
abc123
abc123456789
```

---

# Validation Failure

If validation fails:

```text
Routing Stops
```

and control moves to:

```text
Out Link Error Handling
```

No resolution attempt occurs.

---

# Request Flow

Normal request flow:

```text
Visitor
↓
/out/{public_id}
↓
Routing
↓
Validation
↓
Resolution Layer
```

---

# Routing Output

Successful routing produces:

```json
{
  "public_id": "k7m2x9qp"
}
```

This payload is forwarded to Resolution.

---

# Resolution Contract

Routing must not know:

```text
Destination URL
Profile
Block
Analytics
```

Routing only forwards:

```text
public_id
```

to the Resolution Layer.

---

# HTTP Method Support

Supported:

```text
GET
```

V1 does not support:

```text
POST
PUT
PATCH
DELETE
```

---

# Query Parameters

Routing ignores all query parameters.

Example:

```text
/out/k7m2x9qp?utm=test
```

Routing still resolves:

```text
k7m2x9qp
```

Query handling belongs to later layers.

---

# Route Stability

Once generated:

```text
/out/{public_id}
```

must never change.

The route remains stable throughout the Out Link lifecycle.

---

# Lifecycle Awareness

Routing does not inspect:

```text
Active
Archived
Purged
```

states.

Lifecycle decisions belong to Resolution.

Routing always forwards valid requests.

---

# Security Principles

Routing must never expose:

```text
Internal IDs
Account IDs
Block IDs
Database Structure
```

Only:

```text
public_id
```

is publicly visible.

---

# Logging

Routing may record:

```text
Request Timestamp
Request Path
Validation Result
```

Routing does not record:

```text
Click Analytics
Redirect Analytics
Business Metrics
```

These belong to Tracking.

---

# Design Goal

The Routing Layer should remain:

```text
Simple
Stateless
Fast
Predictable
```

Its only responsibility is moving requests from:

```text
/out/{public_id}
```

to the Resolution Layer.
