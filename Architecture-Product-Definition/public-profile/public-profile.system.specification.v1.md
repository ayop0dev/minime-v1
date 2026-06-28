# Public Profile System Specification V1

## Status

Approved

---

# Purpose

> The Public Profile System is the public rendering surface produced by the Rendering domain. It is not a separate Product Domain or a separate architectural responsibility; it is the public-facing surface of Rendering. This document describes that surface.

The Public Profile System serves Minime profiles to visitors as the public rendering surface produced by Rendering.

It is the public-facing surface of the Rendering domain.

The Public Profile System consumes profile data that has already been processed by Rendering.

Its responsibility is delivery of the rendered surface.

Not content management.

Not profile editing.

---

# Position In Architecture

```text
Account
↓
Username
↓
Profile
↓
Rendering
↓
Public rendering surface (the public profile)
↓
Visitor
```

---

# Responsibilities

The Public Profile System is responsible for:

```text
Profile Resolution

Profile Delivery

Public Requests

Public Error Handling

Caching
```

---

# Not Responsible For

The Public Profile System is NOT responsible for:

```text
Authentication

Account Management

Username Registration

Social Accounts Setup

Profile Editing

Block Editing

Rendering

Theme Management

Layout Management

Analytics Processing

Link Tracking
```

---

# Public Profile Philosophy

The Public Profile System answers:

```text
Which profile should be shown?
```

It does not answer:

```text
What content should exist?

How should content be rendered?

How should themes work?
```

---

# Inputs

The Public Profile System receives:

```text
Profile Request

Username

Rendered Profile Output
```

Example:

```text
https://minime.ae/ahmed
```

---

# Outputs

The Public Profile System produces:

```text
Public Profile Response
```

Example:

```text
Profile Page
```

or

```text
404 Not Found
```

---

# Core Flow

```text
Visitor Request
↓
Routing
↓
Resolution
↓
Access Policy
↓
Load Rendered Output
↓
Return Public Profile
```

---

# Profile Existence Requirement

A profile may be served when:

```text
An account exists

The username resolves

The account is active
```

A profile exists as soon as an account is activated and a valid username is bound to it.

---

# Relationship With Rendering

Public Profile consumes Rendered Profile Output.

Public Profile never:

```text
Builds Themes

Builds Layouts

Builds Render Objects

Transforms Content
```

---

# Relationship With Routing

Routing determines:

```text
Requested Username
```

Public Profile determines:

```text
Profile Response
```

---

# Relationship With Access Policy

Public Profile proceeds to delivery only after the Access Policy allows the request.

Access decisions remain external to this module.

---

# Relationship With Caching

Public Profile may serve cached responses.

Caching must never change profile content.

Caching only improves delivery speed.

---

# Error States

Supported:

```text
Profile Not Found

Account Suspended (Future)

System Error
```

Detailed handling is defined in:

```text
public-profile.error.states.v1.md
```

---

# V1 Scope

Supported:

```text
Single Profile

Username Routing

Public Access

Caching

Access Validation
```

Not Supported:

```text
Subpages

Multi-page Profiles

Password Protected Profiles

Region-based Restrictions
```

---

# Architectural Rule

The Public Profile System is a delivery layer.

It consumes prepared profile output.

It never owns profile content.

It never owns rendering logic.

It never owns presentation logic.

---

# Core Principle

Profile Content stores content.

Rendering generates profile output.

Public Profile delivers the result.
