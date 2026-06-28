# Username System Reserved List V1

## Status

Approved

---

# Purpose

This document defines usernames reserved for Minime platform operations.

These usernames are permanently unavailable for public registration.

System reserved usernames protect:

* Platform routes
* Internal features
* Administrative areas
* Future platform capabilities
* Infrastructure endpoints

---

# Core Principle

System reserved usernames are not public assets.

They exist to protect platform functionality and prevent route conflicts.

All usernames in this list must return:

```text
blocked
```

during username availability checks.

---

# Matching Strategy

System reserved username detection uses:

```text
substring matching
```

Example:

Reserved term:

```text
admin
```

The following usernames are blocked:

```text
admin
admin1
myadmin
adminpanel
superadmin
```

---

# Management

This list is maintained internally by Minime.

Users cannot claim, request, or override these usernames.

---

# Authentication & Access

```text
login
signin
signup
register
logout
auth
authentication
verify
verification
otp
```

---

# Account Management

```text
account
accounts
profile
profiles
settings
security
billing
subscription
subscriptions
plan
plans
upgrade
```

---

# Administrative Functions

```text
admin
administrator
moderator
staff
team
owner
management
console
controlpanel
```

---

# Platform Infrastructure

```text
api
cdn
assets
static
media
storage
files
upload
uploads
download
downloads
```

---

# System Operations

```text
system
internal
private
public
service
services
status
health
monitor
monitoring
logs
```

---

# Website Navigation

```text
home
about
contact
help
support
faq
docs
documentation
blog
news
careers
jobs
```

---

# Legal & Compliance

```text
terms
privacy
cookies
policy
policies
legal
compliance
gdpr
```

---

# Social Accounts

```text
discovery
discover
search
scanner
scan
finder
import
importer
```

---

# Analytics

```text
analytics
stats
statistics
reports
reporting
insights
metrics
tracking
```

---

# Platform Branding

```text
minime
minimeae
official
verified
premium
pro
enterprise
business
```

---

# Future Reserved Categories

The following categories may be expanded in future versions:

* AI features
* Commerce features
* Payments
* Creator programs
* Advertising systems
* Partner systems
* Mobile applications
* Developer tools

Additional reserved terms may be added without modifying the Account Claim System.

---

# Relationship To Other Lists

This document protects platform functionality.

It is separate from:

```text
username.reserved.list.v1.md
```

which protects brands, celebrities, and strategic assets.

And separate from:

```text
username.blocked.terms.v1.md
```

which contains offensive, abusive, or prohibited terms.

---

# Availability Result

Any username matching a system reserved term must return:

```text
blocked
```

during username availability checks.
