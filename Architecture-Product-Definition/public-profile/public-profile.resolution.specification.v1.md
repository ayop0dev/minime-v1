# Public Profile Resolution Specification V1

## Status

Approved

---

# Purpose

The Profile Resolution System is responsible for resolving a username into a specific account and its profile.

It sits between Routing and Public Profile Delivery.

Resolution determines:

```text
Which profile should be loaded?
```

It does not determine:

```text
How the profile is rendered?

How the profile is displayed?

How the profile is delivered?
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
Profile Delivery
```

---

# Responsibilities

The Profile Resolution System is responsible for:

```text
Username Lookup

Account Identification

Profile Retrieval

Resolution Validation
```

---

# Not Responsible For

The Profile Resolution System is NOT responsible for:

```text
Authentication

Rendering

Themes

Layouts

Access Decisions

Caching

Analytics

Tracking

Profile Editing
```

---

# Resolution Input

Input:

```json
{
  "username": "ahmed"
}
```

Provided by:

```text
Routing Layer
```

---

# Resolution Output

Successful resolution returns:

```json
{
  "account_id": "...",
  "username": "ahmed"
}
```

The exact internal Account identifier is implementation specific.

---

# Resolution Philosophy

Resolution answers:

```text
Which profile owns this username?
```

Resolution never answers:

```text
Can the profile be viewed?

Should the profile be shown?

How should the profile look?
```

---

# Username Uniqueness

In V1:

```text
One Username
↓
One Profile
```

A username must resolve to exactly one profile.

---

# Resolution Flow

```text
Receive Username
↓
Lookup Username
↓
Find Profile
↓
Return Profile Reference
```

---

# Successful Resolution

Example:

```text
Request:
↓
/ahmed

Resolution:
↓
Account #123
```

Output:

```json
{
  "account_id": "123",
  "username": "ahmed"
}
```

---

# Failed Resolution

If no matching profile exists:

```text
Resolution Failed
```

Output:

```json
{
  "status": "not_found"
}
```

---

# Reserved Usernames

Reserved usernames must never resolve to public profiles.

Examples:

```text
admin
api
login
register
settings
```

Routing should normally prevent these requests.

Resolution must still protect against them.

---

# Deleted Profiles

If a profile has been deleted:

```text
Resolution Failed
```

Output:

```json
{
  "status": "not_found"
}
```

V1 does not expose deletion status publicly.

---

# Suspended Profiles

Resolution still returns the profile.

Access Policy evaluation occurs downstream.

Example:

```text
Username Exists
↓
Profile Found
↓
Access Policy Decides Whether Request May Be Served
```

---

# Relationship With Routing

Routing provides:

```text
username
```

Resolution provides:

```text
profile reference
```

---

# Relationship With Access Policy

Resolution finds profiles.

Access Policy decides whether the request may be served.

Example:

```text
Resolution
↓
Profile Found
↓
Access Policy Check
↓
Allowed / Denied
```

---

# Relationship With Public Profile System

Resolution does not return UI.

Resolution does not return HTML.

Resolution does not return Presentation Output.

Resolution only returns Account identity.

---

# V1 Scope

Supported:

```text
Username Lookup

Single Profile Resolution

Profile Reference Output
```

Not Supported:

```text
Multiple Profiles Per Username

Alias Usernames

Profile Redirects

Custom Domains

Subpage Resolution
```

---

# Architectural Rule

Resolution identifies profiles.

Access Policy controls access.

Rendering controls appearance.

Delivery controls responses.

These responsibilities must remain separated.

---

# Core Principle

Routing finds the username.

Resolution finds the profile.

Everything after that belongs to downstream systems.
