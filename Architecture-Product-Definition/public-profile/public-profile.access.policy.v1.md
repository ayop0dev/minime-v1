# Public Profile Access Policy V1

## Status

Approved

---

# Purpose

The Access Policy is responsible for determining whether a visitor request may be served.

It sits between:

```text
Profile Resolution
↓
Public Profile Delivery
```

A username may resolve successfully while the request still cannot be served.

---

# Access Policy Philosophy

The Access Policy answers:

```text
Can this request be served?
```

It does not answer:

```text
Which profile should be loaded?

How should the profile be rendered?

How should the profile be displayed?
```

---

# Position In Architecture

```text
Routing
↓
Profile Resolution
↓
Access Policy
↓
Public Profile Delivery
```

---

# Responsibilities

The Access Policy is responsible for:

```text
Request Validation

Account Existence Check

Username Resolution Check

Reserved Username Check

Account Status Check

System Integrity Check
```

---

# Not Responsible For

The Access Policy is NOT responsible for:

```text
Username Resolution

Rendering

Themes

Layouts

Caching

Analytics

Tracking

Profile Editing
```

---

# Access Validation Conditions

The Access Policy evaluates whether:

```text
The account exists

The username resolves to an account

The username is not reserved

The account has not been deleted

The account is not suspended

The request is well-formed
```

Each condition must pass before delivery proceeds.

---

# Access Outcomes

The Access Policy produces exactly one of:

```text
Allow
```

or

```text
Deny
```

---

# Allow

Definition:

```text
All access conditions are satisfied.
```

Result:

```text
Proceed to Profile Delivery
```

---

# Deny

Definition:

```text
One or more access conditions are not satisfied.
```

Result:

```text
Return 404 Not Found
```

The visitor is not informed of the specific reason.

---

# Deny Conditions

## Account Does Not Exist

The username does not correspond to any account.

Result:

```text
Deny → 404
```

---

## Username Does Not Resolve

The username cannot be resolved to an account.

Result:

```text
Deny → 404
```

---

## Reserved Username

The requested username is reserved for system use.

Result:

```text
Deny → 404
```

---

## Deleted Account

The account associated with the username has been deleted.

Result:

```text
Deny → 404
```

---

## Suspended Account

The account status is `suspended`.

Result:

```text
Deny → 404
```

---

## Malformed Request

The request cannot be parsed into a valid username or route.

Result:

```text
Deny → 404
```

---

## System Failure

An unexpected system error prevents access validation from completing.

Result:

```text
500 Internal Server Error
```

---

# Access Flow

```text
Request Received
↓
Account Exists?
↓
Yes → Username Resolves?
↓
Yes → Not Reserved?
↓
Yes → Account Not Deleted?
↓
Yes → Account Not Suspended?
↓
Yes → Request Well-Formed?
↓
Yes → Allow
No at any step → Deny
```

---

# Public Privacy Principle

Visitors must not be able to determine:

```text
Whether the account never existed

Whether the account was deleted

Whether the account is suspended

Why the request was denied
```

All deny states produce the same public outcome:

```text
404 Not Found
```

---

# Access Policy Output

Allow:

```json
{
  "allowed": true
}
```

Deny:

```json
{
  "allowed": false
}
```

---

# Relationship With Resolution

Resolution determines:

```text
Account Found
```

Access Policy determines:

```text
Request Serviceable
```

---

# Relationship With Delivery

The Access Policy provides:

```text
Allow
```

or

```text
Deny
```

The delivery layer determines the final response.

---

# V1 Scope

Supported:

```text
Account Existence Check

Username Resolution Check

Reserved Username Check

Deleted Account Check

Suspended Account Check

Well-Formed Request Check

System Failure Handling
```

Not Supported:

```text
Password Protected Profiles

Follower-only Profiles

Regional Restrictions

Age Restrictions

IP Restrictions
```

---

# Architectural Rule

The Access Policy determines access.

The Access Policy never:

```text
Loads Profiles

Renders Profiles

Builds UI

Transforms Content
```

---

# Core Principle

A username may resolve.

A request may still be denied.

The Access Policy is the gate between resolution and delivery.
