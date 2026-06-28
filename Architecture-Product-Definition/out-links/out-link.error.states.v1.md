# Out Link Error States Specification V1

## Status

Approved

---

# Purpose

The Error States Layer defines all supported failure outcomes within the Out Link System.

Its purpose is to provide predictable behavior whenever a redirect request cannot be completed.

---

# Scope

This specification defines:

```text
Error Types
Error Codes
System Responses
Lifecycle Outcomes
```

This specification does not define:

```text
UI Design
Page Layout
Themes
Visual Styling
```

Public profile response presentation decisions belong to the Public Profile System.

---

# Core Principle

Out Link failures should be:

```text
Simple
Predictable
Deterministic
```

Every failure must resolve into a known error state.

---

# Supported Error States

V1 supports only:

```text
INVALID_IDENTIFIER
LINK_NOT_FOUND
LINK_ARCHIVED
```

No additional error states exist.

---

# Error State: INVALID_IDENTIFIER

## Description

The request does not contain a valid Out Link identifier.

Example:

```text
/out
/out/
/out/abc
/out/ABC12345
/out/abc-1234
```

Identifier validation fails before resolution begins.

---

## Trigger

Occurs when:

```text
Identifier Missing
Identifier Length Invalid
Identifier Format Invalid
```

---

## Lifecycle Outcome

```text
Routing Stops
Resolution Not Started
No Redirect
No Analytics Event
```

---

## Error Code

```text
INVALID_IDENTIFIER
```

---

# Error State: LINK_NOT_FOUND

## Description

The identifier is valid but no matching Out Link exists.

Example:

```text
/out/k7m2x9qp
```

where:

```text
public_id
```

cannot be found.

---

## Trigger

Occurs when:

```text
Resolution Lookup Returns No Record
```

---

## Lifecycle Outcome

```text
Resolution Stops
No Redirect
No Analytics Event
```

---

## Error Code

```text
LINK_NOT_FOUND
```

---

# Error State: LINK_ARCHIVED

## Description

The Out Link exists but is no longer active.

Example:

```text
status = archived
```

---

## Trigger

Occurs when:

```text
Resolution Finds Archived Record
```

---

## Lifecycle Outcome

```text
Resolution Stops
No Redirect
No Analytics Event
```

---

## Error Code

```text
LINK_ARCHIVED
```

---

# Purged Records

Purged records do not generate a unique error state.

Reason:

```text
Purged Records No Longer Exist
```

---

Behavior:

```text
Purged Record
↓
No Record Found
↓
LINK_NOT_FOUND
```

---

# Error Handling Contract

Every failure outcome returns:

```text
One Error State
```

Only.

No multiple-error responses exist.

---

# Analytics Rules

Error states never create:

```text
Click Events
Analytics Events
```

Errors are not considered successful link activity.

---

# Redirect Rules

Error states never produce:

```text
302 Redirect
307 Redirect
```

Redirect execution stops immediately.

---

# Presentation Contract

The Out Link System returns:

```text
Error Code
```

Only.

The system does not decide:

```text
Messaging
Layout
Language
Visual Appearance
```

These decisions belong to the Public Profile System.

---

# Error Mapping

```text
INVALID_IDENTIFIER
↓
Invalid Link

LINK_NOT_FOUND
↓
Link Not Found

LINK_ARCHIVED
↓
Link No Longer Available
```

These mappings are recommendations only.

Presentation layers may choose different wording.

---

# Logging

The system may record:

```text
Timestamp
Error Code
Request Path
```

for operational purposes.

Error logging must not generate analytics events.

---

# V1 Error Matrix

| Error State        | Redirect | Analytics Event |
| ------------------ | -------- | --------------- |
| INVALID_IDENTIFIER | No       | No              |
| LINK_NOT_FOUND     | No       | No              |
| LINK_ARCHIVED      | No       | No              |

---

# Design Goal

The Error States Layer should remain:

```text
Small
Predictable
Stable
```

Every failed Out Link request must resolve to exactly one of:

```text
INVALID_IDENTIFIER
LINK_NOT_FOUND
LINK_ARCHIVED
```

and nothing else.
