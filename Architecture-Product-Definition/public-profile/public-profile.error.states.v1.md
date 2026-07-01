# Public Profile Error States Specification V1

## Status

Approved

---

# Purpose

This document defines all public-facing error states for Minime V1 profile requests.

Its purpose is to ensure:

```text
Consistency

Predictability

Privacy

Security
```

across all public profile responses.

---

# Error Philosophy

Visitors should receive simple and predictable responses.

Internal system details must never be exposed.

Profile status information must remain private.

---

# Supported Public Responses

V1 supports:

```text
200 OK

404 Not Found

500 Internal Server Error
```

Only.

---

# 200 OK

Definition:

```text
Profile exists.

Profile resolves successfully.

Access is permitted.
```

Result:

```text
Serve Public Profile
```

---

# 404 Not Found

Definition:

Returned whenever the profile cannot be publicly served.

The visitor must not know why.

---

# 404 Scenarios

## Username Does Not Exist

Example:

```text
/unknown-user
```

Result:

```text
404 Not Found
```

---

## Profile Deleted

Example:

```text
Profile existed previously.
```

Result:

```text
404 Not Found
```

---

## Account Suspended

Example:

```text
Account access disabled by the system.
```

Result:

```text
404 Not Found
```

---

## QR Code Cannot Resolve

Example:

```text
/qr/{qr_code_id} where the QR Code record does not exist, belongs to a deleted Account, or cannot resolve to an active public profile.
```

Result:

```text
404 Not Found
```

The QR redirect route must never expose internal Account data while failing.

---

# Privacy Rule

Visitors must not be able to distinguish between:

```text
Never Existed

Deleted

Suspended
```

All produce:

```text
404 Not Found
```

---

# 500 Internal Server Error

Definition:

An unexpected system failure prevents the request from completing.

Examples:

```text
Database Failure

Service Failure

Unexpected Exception

Infrastructure Failure
```

---

# 500 Response Policy

The public response must never expose:

```text
Stack Traces

Database Information

Internal IDs

System Details

Implementation Details
```

---

# Example Public Response

Allowed:

```text
Something went wrong.
Please try again later.
```

Not allowed:

```text
Database connection failed.

SQL Exception.

Profile table missing.
```

---

# Relationship With Routing

Routing failures return:

```text
404 Not Found
```

when no valid profile route can be resolved.

---

# Relationship With Resolution

Resolution failures return:

```text
404 Not Found
```

when no profile can be identified.

---

# Relationship With Access Policy

Access Policy denials return:

```text
404 Not Found
```

regardless of the reason.

---

# Relationship With Delivery

The delivery layer determines:

```text
HTTP Status Code

Response Body

Response Headers
```

based on the error state.

---

# Error Flow

```text
Request
↓
Routing
↓
Resolution
↓
Access Policy
↓
Success
→ 200

Failure
→ 404

System Error
→ 500
```

---

# V1 Scope

Supported:

```text
200 Responses

404 Responses

500 Responses
```

Not Supported:

```text
401 Unauthorized

403 Forbidden

429 Rate Limited

451 Restricted Content
```

for public profile viewing.

---

# Architectural Rule

Error responses must reveal the minimum amount of information necessary.

Internal state must remain private.

Public responses must remain consistent.

---

# Core Principle

If a profile cannot be served:

Return 404.

If the system fails:

Return 500.

Nothing more.
