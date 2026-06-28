# Username Blocked Terms V1

## Status

Approved

---

# Purpose

This document defines prohibited username terms that are not allowed on Minime.

These terms are blocked to protect:

* Platform integrity
* User safety
* Brand reputation
* Legal compliance
* Community standards

Any username matching a blocked term must return:

```text
blocked
```

during username availability checks.

---

# Core Principle

Blocked terms are prohibited regardless of availability.

Example:

```text
fuck
```

returns:

```text
blocked
```

even if the username has never been claimed.

---

# Matching Strategy

Blocked term detection uses:

```text
substring matching
```

Example:

Blocked term:

```text
fuck
```

The following usernames are blocked:

```text
fuck
fuck1
myfuck
officialfuck
fuck2026
```

---

# User Visibility

Users must never be informed which blocked term caused rejection.

The system only returns:

```text
blocked
```

---

# Management

This list is maintained internally by Minime.

Terms may be added, removed, or updated without modifying the Account Claim System.

---

# Adult & Sexual Content

Examples:

```text
sex
porn
porno
xxx
adult
nude
naked
escort
camgirl
camboy
onlyfans
```

---

# Explicit Profanity

Examples:

```text
fuck
fucking
shit
bullshit
bitch
asshole
motherfucker
bastard
dick
pussy
cunt
slut
whore
```

---

# Hate & Abuse

Examples:

```text
racist
terrorist
nazism
nazi
hitler
genocide
extremist
```

---

# Violence & Criminal Activity

Examples:

```text
murder
killer
killerforhire
assassin
terror
bomb
explosive
cartel
druglord
```

---

# Fraud & Scam

Examples:

```text
scam
fraud
fakeaccount
phishing
hacker
hacking
cracker
carding
stealer
```

---

# Illegal Substances

Examples:

```text
cocaine
heroin
meth
methamphetamine
ecstasy
drugdealer
```

---

# Self-Harm

Examples:

```text
suicide
selfharm
killmyself
cutting
```

---

# Impersonation Terms

Examples:

```text
supportteam
customersupport
officialsupport
verificationteam
securityteam
```

These terms are blocked to reduce impersonation attempts.

---

# Misleading Authority Terms

Examples:

```text
adminteam
staffteam
modteam
officialaccount
verifiedaccount
```

---

# Reserved Future Expansion

Additional categories may be added:

* Emerging abuse terms
* Regional profanity
* Country-specific prohibited terms
* Regulatory requirements
* Platform abuse patterns

---

# Relationship To Other Lists

This document contains prohibited terms.

It is separate from:

```text
username.reserved.list.v1.md
```

which protects brands, celebrities, organizations, and strategic assets.

And separate from:

```text
username.system.reserved.list.v1.md
```

which protects platform functionality and internal routes.

---

# Availability Result

Any username matching a blocked term must return:

```text
blocked
```

during username availability checks.
