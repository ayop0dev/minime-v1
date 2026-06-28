# Username Policy V1

## Status

Approved

---

# Purpose

This document defines the official username rules used throughout Minime.

All systems that accept, validate, check, create, reserve, or manage usernames must follow this policy.

This document is the single source of truth for username behavior.

---

# Scope

Applies to:

* Account Claim System
* Username Availability Checks
* Username Reservations
* Account Creation
* Public Profile URLs
* Future Username Management Features

---

# Username Format

A username represents the public identity of a Minime profile.

Example:

```text
minime.ae/ahmed
```

Where:

```text
ahmed
```

is the username.

---

# Allowed Characters

Usernames may contain only:

```text
a-z
0-9
```

Allowed examples:

```text
ahmed
hala
ahmed2026
minime
creator1
```

---

# Disallowed Characters

The following characters are not permitted:

```text
-
_
.
@
#
$
%
&
+
=
/
\
?
!
,
:
;
(
)
[
]
{
}
'
"
```

Spaces are not allowed.

---

# Non-Latin Characters

Usernames may not contain:

```text
Arabic letters
Chinese characters
Japanese characters
Emoji
Unicode symbols
```

Examples:

```text
أحمد
محمد123
😀
```

are invalid.

---

# Username Length

Minimum length:

```text
3
```

Maximum length:

```text
30
```

---

# Valid Examples

```text
ahmed
hala
creator
ahmed2026
builder1
mohamedhassan
```

---

# Invalid Examples

```text
ah
```

Reason:

```text
Below minimum length
```

---

```text
ahmed-salem
```

Reason:

```text
Contains disallowed character
```

---

```text
ahmed_salem
```

Reason:

```text
Contains disallowed character
```

---

```text
ahmed.salem
```

Reason:

```text
Contains disallowed character
```

---

```text
أحمد
```

Reason:

```text
Non-Latin characters
```

---

# Username Normalization

All usernames must be normalized before validation.

Normalization steps:

```text
Trim whitespace
Convert to lowercase
```

Example:

```text
Ahmed
AHMED
aHmEd
```

becomes:

```text
ahmed
```

---

# Case Sensitivity

Usernames are case-insensitive.

Examples:

```text
Ahmed
AHMED
ahmed
```

represent the same username.

Storage format:

```text
lowercase only
```

Display format:

```text
lowercase only
```

---

# Username Availability Statuses

Only three availability statuses exist.

## available

The username may be claimed immediately.

Example:

```text
ahmed
```

is not owned and not blocked.

---

## taken

The username is currently unavailable.

This includes:

* Existing approved accounts
* Active username reservations

Users must not be informed which condition applies.

---

## blocked

The username is prohibited by Minime.

This includes:

* Premium Reserved Names
* System Reserved Names
* Blocked Terms

---

# Availability Check Rules

Availability checks begin only after the username reaches the minimum length.

Example:

```text
a
```

No availability check.

---

```text
ah
```

No availability check.

---

```text
ahm
```

Availability check allowed.

---

# Blocked Username Detection

Blocked username evaluation occurs before availability checks.

Validation order:

```text
Normalize
↓
Validate Format
↓
Check Block Lists
↓
Check Availability
```

---

# Block Lists

Three independent block lists exist, all defined in:

```text
username.reserved.and.blocked.lists.v1.md
```

- **System Reserved** — platform routes and internal functionality (e.g. `admin`, `api`, `login`, `signup`).
- **Brand & Identity Reserved** — brands, public figures, organizations (e.g. `nike`, `google`, `messi`, `cristiano`).
- **Blocked Terms** — offensive, abusive, illegal, or otherwise restricted terms.

---

# Block Matching Strategy

Block matching uses substring matching.

Example:

Blocked term:

```text
nike
```

The following usernames are blocked:

```text
nike
nike1
mynike
officialnike
nike2026
```

---

# Username Uniqueness

Usernames must be globally unique.

Example:

```text
ahmed
```

and

```text
ahmed
```

cannot both exist.

---

The following are different usernames:

```text
ahmed
ahmed1
ahmed01
ahmed2026
```

and may coexist.

---

# Future Versions

Future versions may introduce:

* Username changes
* Username recovery
* Username redirects
* Additional moderation rules

These features are outside the scope of V1.
