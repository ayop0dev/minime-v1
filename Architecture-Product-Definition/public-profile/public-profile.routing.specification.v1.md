# Public Profile Routing Specification V1

## Status

Approved

---

# Purpose

This document defines how public profile URLs are routed in Minime V1.

Routing is responsible for extracting the requested username from an incoming URL.

Routing does not load profiles.

Routing does not validate profiles.

Routing does not render profiles.

Routing only resolves requests into route parameters.

---

# Routing Philosophy

Routing answers:

```text
Which username was requested?
```

Routing does not answer:

```text
Does the profile exist?

Can the profile be viewed?
```

Those responsibilities belong to other systems.

---

# Primary Public Route

V1 supports:

```text
https://minime.ae/{username}
```

Example:

```text
https://minime.ae/ahmed
https://minime.ae/hala
https://minime.ae/omar
```

---

# Route Parameter

The route parameter is:

```text
username
```

Example:

```text
/ahmed
```

returns:

```json
{
  "username": "ahmed"
}
```

---

# Reserved System Routes

The following routes are reserved for the platform:

```text
/
/login
/register
/social-accounts
/dashboard
/settings
/pricing
/help
/privacy
/terms
/admin
/api
/out
/qr
```

These routes must never be interpreted as usernames.

---

# Username Routing Priority

Routing order:

```text
Reserved System Route
↓
Username Route
↓
Not Found
```

Example:

```text
/settings
```

must resolve to:

```text
System Route
```

not:

```text
Username = settings
```

---

# Username Validation

The routing layer may perform basic username format validation.

Examples:

Valid:

```text
ahmed
hala
omar123
```

Invalid:

```text
ahmed/
hello world
@@@
```

Full username rules are defined elsewhere.

---

# Routing Output

Successful routing produces:

```json
{
  "type": "profile",
  "username": "ahmed"
}
```

---

# Unsupported Routes

V1 does not support:

```text
/{username}/about

/{username}/links

/{username}/projects

/{username}/{page}
```

---

# Subpages

Not supported in V1.

Examples:

```text
/ahmed/about
/ahmed/store
/ahmed/contact
```

must return:

```text
404
```

---

# Query Parameters

Query parameters may exist.

Example:

```text
/ahmed?utm_source=instagram
```

Routing still resolves:

```json
{
  "username": "ahmed"
}
```

Query parameters do not affect routing decisions.

---

# QR Code Redirect Route

V1 supports exactly one additional public route for QR Code resolution:

```text
https://minime.ae/qr/{qr_code_id}
```

This route is owned by the Account domain's QR Code System (`Architecture-Product-Definition/qr-code/qr-code.system.specification.v1.md`), not by the username routing model.

Rules:

```text
The route parameter is qr_code_id, never username.
qr_code_id must resolve the QR Code record.
The QR Code record must resolve account_id.
account_id must resolve the Account's current username by reading the Account record live — never from a value stored on the QR Code record.
The route must redirect (HTTP redirect) to /{username}.
The route must never render a separate public profile page.
The route must never expose internal Account data.
```

The canonical resolution flow is:

```text
qr_code_id
↓
AccountQRCode
↓
account_id
↓
Account.username
↓
Redirect to /{username}
```

The QR Code record never stores `username` or a username snapshot; this flow always reads the current value from the Account record.

If the QR Code record does not exist, belongs to a deleted Account, or cannot resolve to an active public profile, the route must fail safely per `public-profile.error.states.v1.md` — it must never redirect to a partial, internal, or non-existent destination.

This route must never be interpreted as a username request, and `qr_code_id` must never be confused with `username`.

---

# Relationship With Resolution

Routing extracts:

```text
username
```

Profile Resolution determines:

```text
which profile belongs to that username
```

---

# Relationship With Public Profile System

Routing provides:

```text
Route Parameters
```

The Public Profile System provides:

```text
Profile Response
```

---

# V1 Scope

Supported:

```text
Single Username Route

Reserved Routes

Query Parameters

Username Resolution Input
```

Not Supported:

```text
Subpages

Multi-page Profiles

Custom URL Structures

Nested Routes

Localized Routes
```

---

# Architectural Rule

Routing is a request parser.

Routing never:

```text
Loads Profiles

Checks Access

Builds UI

Returns Profile Data
```

---

# Core Principle

Routing identifies the requested username.

Everything after that belongs to the Public Profile System.
