# Minime Account Claim System Specification V1

## Status

Approved

---

# Purpose

The Minime Account Claim System is responsible for creating and claiming Minime user accounts.

Its responsibility begins when a visitor enters a desired Minime username and ends when an approved user account and public profile have been successfully created.

The Account Claim System is completely independent from Social Accounts Setup.

Social Accounts Setup is a later onboarding step and does not participate in username selection, username ownership, account registration, authentication, or account approval.

---

# Scope

The Account Claim System is responsible for:

* Username selection
* Username validation
* Username availability checks
* Username reservation
* User authentication
* User verification
* Account creation
* Public profile creation

The Account Claim System is not responsible for:

* Social account collection and normalization
* Social account ownership
* Social account verification
* Profile data import
* Analytics
* Personal branding features

---

# Core Principles

## Username First

Minime is username-first.

The onboarding flow begins with:

```text
minime.ae/{username}
```

before any authentication occurs.

Users claim a username first, then authenticate ownership of the account.

---

## One User = One Username = One Profile

V1 supports a single public profile per account.

```text
1 User
=
1 Username
=
1 Public URL
=
1 Public Profile
```

Multiple profiles are out of scope for V1.

---

## Usernames Are Platform Independent

Minime usernames are completely independent from social media usernames.

Examples:

```text
Minime Username:
ahmed

Instagram:
ahmedofficial

TikTok:
ahmedtv

YouTube:
ahmedchannel
```

No relationship is required between them.

---

## Social Accounts Setup Is Post Registration

Social Accounts Setup runs only after successful account creation.

Account Claim System must never depend on Social Accounts Setup.

---

# Username Policy

Username rules are defined by:

```text
username.policy.v1.md
```

The Account Claim System must use those rules when validating usernames.

---

# Username Availability

A username may return one of the following statuses:

## available

The username may be claimed immediately.

## taken

The username is currently unavailable.

This includes:

* Existing approved accounts
* Active username reservations

## blocked

The username is prohibited by Minime policies.

This includes:

* Premium reserved names
* System reserved names
* Blocked terms

---

# Username Normalization

All usernames must be normalized before validation and availability checks.

Rules:

```text
Convert to lowercase
Trim whitespace
```

Examples:

```text
Ahmed
AHMED
ahmed
```

becomes:

```text
ahmed
```

---

# Username Reservation

## Purpose

Username reservations temporarily protect a username during account creation.

This prevents race conditions between multiple users attempting to claim the same username.

---

## Reservation Creation

A reservation is created after:

```text
Username Available
↓
Claim Started
```

---

## Reservation Duration

```text
10-15 minutes
```

Implementation may choose an exact value within this range.

---

## Reservation Visibility

Reservations are internal only.

Users never see:

```text
reserved
```

Reservations are exposed as:

```text
taken
```

during availability checks.

---

## Reservation Expiry

If account creation is not completed:

```text
Reservation Expires
↓
Username Returns To Available
```

automatically.

No recovery mechanism exists in V1.

---

# Authentication Methods

Authentication rules are defined by:

```text
authentication.policy.v1.md
```

Supported methods:

## Email OTP

User enters email address.

A one-time password is delivered by email.

Successful OTP verification completes authentication.

---

## Google Sign-In

User authenticates using Google.

Successful authentication is considered verified immediately.

No additional OTP is required.

---

## Unsupported In V1

The following methods are out of scope:

```text
Phone Signup
Phone OTP
SMS Verification
```

Reason:

```text
Variable Cost ≈ 0
```

is a core project requirement.

---

# Account Approval

Account approval occurs immediately after successful authentication.

Examples:

```text
Username Reserved
↓
Email OTP Verified
↓
Approved User Account
```

or

```text
Username Reserved
↓
Google Authentication
↓
Approved User Account
```

---

# Account Identity

Every approved account must receive a permanent internal identifier.

Example:

```text
acc_01JXXXX
```

Requirements:

* Globally unique
* Permanent
* Never changes
* Independent from username

---

# Username Ownership

Approved accounts own their usernames.

Example:

```text
acc_01JXXXX
↓
ahmed
```

V1 does not support username changes.

Username change policies may be introduced in future versions.

---

# Public Profile Creation

Immediately after account approval:

```text
Approved User Account
↓
Create Public Profile
```

The public profile exists immediately even if it contains no content.

Example:

```text
minime.ae/ahmed
```

---

# Registration Flow

```text
Enter Username
↓
Normalize Username
↓
Validate Username
↓
Check Availability
↓
Available
↓
Create Username Reservation
↓
Choose Authentication Method
↓
Email OTP
OR
Google Sign-In
↓
Successful Authentication
↓
Create Approved User Account
↓
Attach Username
↓
Delete Reservation
↓
Create Public Profile
↓
Welcome Screen
↓
Launch Social Accounts Setup
```

---

# Social Accounts Setup Handoff

After successful account creation:

```text
Welcome Screen
↓
Social Accounts Setup
```

The user may:

```text
Add Social Accounts
```

or

```text
Skip To Dashboard
```

Social Accounts Setup remains optional.

---

# Related Documents

```text
username.policy.v1.md
authentication.policy.v1.md
username.reserved.and.blocked.lists.v1.md
```
