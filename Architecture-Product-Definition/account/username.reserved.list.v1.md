# Username Reserved List V1

## Status

Approved

---

# Purpose

This document defines usernames that are permanently unavailable for public registration.

These usernames are reserved by Minime and cannot be claimed through normal account creation.

Reserved usernames are treated as:

```text
blocked
```

during username availability checks.

---

# Purpose Of Reserved Names

Reserved names exist to protect:

* Global brands
* Major companies
* Public figures
* Celebrities
* Sports organizations
* Governments
* Countries
* High-value digital assets
* Future Minime strategic assets

---

# Core Principle

Reserved usernames are not available to the public.

Ownership verification is not performed in V1.

The existence of a reserved username in this list automatically blocks registration.

Example:

```text
nike
```

returns:

```text
blocked
```

for all users.

---

# Matching Strategy

Reserved username detection uses:

```text
substring matching
```

Example:

Reserved term:

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
bestnike
```

---

# Management

This list is managed internally by Minime.

Users cannot appeal, request, or override reserved usernames in V1.

---

# Categories

## Global Brands

Examples:

```text
nike
adidas
apple
google
microsoft
amazon
meta
facebook
instagram
threads
tiktok
youtube
spotify
netflix
tesla
openai
chatgpt
```

---

## Major Technology Companies

Examples:

```text
samsung
intel
amd
nvidia
oracle
ibm
cisco
paypal
stripe
shopify
```

---

## Global Public Figures

Examples:

```text
messi
cristiano
ronaldo
neymar
mbappe
```

---

## International Celebrities

Examples:

```text
taylorswift
kimkardashian
therock
mrbeast
```

---

## Arab Celebrities

Examples:

```text
ahmedelsakka
mohamedramadan
amrdiab
tamerhosny
adelimam
```

---

## Sports Organizations

Examples:

```text
fifa
uefa
laliga
premierleague
nba
```

---

## News Organizations

Examples:

```text
bbc
cnn
reuters
aljazeera
skynews
```

---

## Government Entities

Examples:

```text
whitehouse
pentagon
nasa
unesco
unicef
```

---

## Countries

Examples:

```text
egypt
saudiarabia
uae
usa
france
germany
```

---

## High-Value Generic Names

Examples:

```text
news
media
sports
music
movies
tv
radio
official
verified
premium
vip
```

---

# V1 Strategy

The initial list should contain a curated collection of high-value usernames.

The list is expected to grow continuously over time.

Expansion of the list does not require changes to the Account Claim System.

Only this document requires updates.

---

# Relationship To Other Lists

Reserved usernames are separate from:

```text
username.system.reserved.list.v1.md
```

which protects Minime infrastructure.

And separate from:

```text
username.blocked.terms.v1.md
```

which contains offensive or prohibited terms.

---

# Availability Result

Any username matching a reserved term must return:

```text
blocked
```

during username availability checks.
