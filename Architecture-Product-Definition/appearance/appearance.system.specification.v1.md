# Appearance System Specification V1

## Status

Approved

---

# Purpose

This document defines the Appearance system of Minime V1.

Appearance is the design area of the Profile domain responsible for the visual configuration of a profile.

It defines how Profile Content should appear.

It never defines what Profile Content contains.

Appearance is part of the Profile domain within the Minime V1 Product Architecture. It is the design area of the Profile domain, not a standalone domain.

It participates in profile rendering by providing visual configuration, but it never renders content itself.

---

# Philosophy

Minime intentionally separates content from presentation.

This separation is a permanent architectural principle.

The system answers three different questions through three different domains.

Profile Content answers:

```text
What should be shown?
```

Appearance answers:

```text
How should it look?
```

Rendering answers:

```text
How is the final public profile produced?
```

These responsibilities never overlap.

---

# Architecture Position

```text
[Conceptual Architecture — Domain Ownership]

Account
        │
        ▼
Profile Content
├── Profile Data
├── Blocks
└── Appearance
        │
        ▼
Rendering
        │
        ▼
Public Profile
```

Appearance is conceptually part of Profile Content.

Appearance is not part of:

* Account
* Blocks
* Rendering
* Public Profile

Its responsibility begins after content exists and ends before rendering begins.

---

# Domain Mission

The mission of the Appearance system is to maintain a consistent visual configuration for every profile.

Appearance guarantees that visual decisions remain independent from:

* profile data
* block content
* rendering implementation
* account management

This separation keeps Minime simple, predictable, and extensible.

---

# Domain Responsibility

Appearance is responsible for every decision that affects the visual presentation of a profile.

Its responsibility is limited to visual configuration.

Appearance never owns profile content itself.

Appearance never owns runtime execution.

Appearance never owns visual production.

Those responsibilities belong to Rendering.

---

# Core Responsibilities

Appearance is responsible for:

* Theme Library integration
* Theme selection
* Theme customization
* Theme constraints
* Appearance State
* Visual configuration
* Visual consistency
* Appearance validation

Appearance is responsible only for visual configuration.

---

# What Appearance Is Responsible For

```text
[Appearance — Responsibility Areas]

Appearance
│
├── Theme Library
│
├── Theme Selection
│
├── Theme Customization
│
├── Theme Constraints
│
└── Appearance State
```

These are the only responsibilities handled by the Appearance system.

---

# What Appearance Never Owns

Appearance never owns:

```text
Profile Data

Blocks

Block Content

Block Ordering

Rendering

Rendering Engine

Layout

Public Profile

Analytics

Out Links

Social Accounts

Authentication

Username

Connected Accounts

Settings

QR Code
```

These responsibilities belong to their respective domains.

---

# User-Facing Representation

Appearance is an internal architectural domain.

Users never interact directly with Appearance.

Instead, users interact with three product sections:

```text
Content

Design

Settings
```

The Design editor is the user-facing representation of the Appearance system.

Users edit Design.

The system produces the resulting Appearance State.

---

# Relationship With Profile Content

```text
[Conceptual Architecture — Domain Ownership]

Profile Content
├── Profile Data
├── Blocks
└── Appearance
```

This diagram represents ownership only.

It does not describe storage.

It does not describe runtime.

---

# Relationship With Persistence

```text
[Persistence View — Storage Structure]

Profile Content Record
│
├── Profile Data
│
└── Appearance State
```

Appearance State is persisted as a section inside the Profile Content record.

Appearance does not own an independent persisted entity.

This architecture follows the Domain Entity Canon.

---

# Relationship With Theme Library

Theme Library is a subsystem of the Appearance system.

It provides:

* Theme Catalog
* Theme Definitions
* Theme Selection Rules
* Theme Customization Rules
* Theme Constraints

Theme Library is a platform-managed supporting resource.

It is never owned by an account.

It is never copied into Profile Content.

Only the following become part of the Appearance State:

* selected_theme_id
* resolved visual configuration
* user customization values

Theme Definitions always remain inside the Theme Library and are resolved at runtime.

---

# Relationship With Rendering

Appearance never renders anything.

Appearance prepares visual configuration.

Rendering consumes that configuration.

Canonical relationship:

```text
[Conceptual Architecture — Domain Ownership]

Profile Content
        +
Appearance
        │
        ▼
Rendering
        │
        ▼
Public Profile
```

Appearance never:

* generates HTML
* generates CSS
* produces Render Objects
* performs layout
* assembles pages

Those responsibilities belong exclusively to the Rendering Domain.

---

# Relationship With Blocks

Profile owns profile content and presentation.

Within the Profile domain, blocks participate in placement and rendering, while the Appearance system handles visual configuration.

Blocks never own:

* themes
* global colors
* typography
* global backgrounds
* spacing system
* visual configuration

Blocks simply expose content.

Rendering combines:

* Block content
* Appearance State
* Theme Library

to produce the final visual result.

---

# Relationship With Settings

Appearance is not Settings.

Settings store account preferences.

Appearance stores profile visual configuration.

No Appearance configuration should be stored inside Settings.

The two domains have independent responsibilities.

---

# Relationship With Account

Account owns identity.

Appearance is responsible for presentation.

The ownership chain is:

```text
account_id
        │
        ▼
Profile Content
        │
        ▼
Appearance State
```

Appearance never owns:

* account identity
* authentication
* ownership
* account lifecycle

Account remains the aggregate root of the system.

---

# Non-Goals

The Appearance system is intentionally limited.

Appearance is **not**:

* a website builder
* a page builder
* a layout editor
* a rendering engine
* a template engine
* a content editor
* a publishing workflow
* a runtime system
* a page composition engine

Appearance is responsible only for profile visual configuration.

Any responsibility outside that boundary belongs to another domain.

# Architecture Principles

The Appearance system follows all approved Minime V1 Architecture Canons.

These principles are permanent and apply to every future Appearance specification.

---

## Principle 1 — Content And Appearance Are Independent

Content and Appearance are independent architectural concerns.

Changing content must never require changing Appearance.

Changing Appearance must never modify content.

Example:

```text
User changes:

- Theme
- Colors
- Background

↓

Bio
Buttons
Links
Avatar

remain unchanged.
```

Likewise:

```text
User edits:

Bio

↓

Theme
Typography
Colors

remain unchanged.
```

---

## Principle 2 — Appearance Is Stateless At Runtime

Appearance stores visual configuration only.

Appearance performs no runtime processing.

At request time the Rendering Domain consumes the persisted Appearance State.

Appearance itself executes nothing.

---

## Principle 3 — One Appearance Per Profile

Each profile owns exactly one Appearance State.

There are:

* no multiple appearances
* no active/inactive appearances
* no appearance versions
* no appearance history
* no appearance branching

V1 always has one active visual configuration.

---

## Principle 4 — Always Live

Appearance follows the global Minime publishing philosophy.

There is:

* no publish button
* no drafts
* no preview mode
* no staging appearance
* no live/draft split

Canonical flow:

```text
User edits Design

↓

Appearance State updates

↓

Persist

↓

Immediately Live
```

---

## Principle 5 — Theme Is A Starting Point

Themes provide platform defaults.

Appearance stores the resolved visual configuration.

A Theme is never the final visual state.

Canonical flow:

```text
Theme Definition

↓

Default Values

↓

User Customization

↓

Appearance State
```

---

## Principle 6 — Rendering Owns Output

Appearance never produces:

* HTML
* CSS
* Render Objects
* Components
* Layout
* Public Pages

Rendering owns all output generation.

---

# Appearance State

Appearance State represents the complete visual configuration of one profile.

It is the output produced by the Design editor.

Appearance State belongs to the Appearance system.

It is persisted inside Profile Content.

It is never an independent entity.

---

## Appearance State Responsibilities

Appearance State stores the resolved visual configuration for one profile.

Its responsibilities include:

* active theme selection
* visual customization values
* validated visual configuration
* profile-specific appearance preferences

Appearance State does not contain Theme Definitions.

Appearance State references Theme Library resources.

---

## Appearance State May Contain

The exact schema is defined in dedicated specifications.

Typical values include:

```text
selected_theme_id

background

color configuration

typography configuration

button configuration

corner radius

shadow configuration

spacing configuration

animation preferences
```

This document intentionally avoids defining field-level implementation.

Its responsibility is architectural ownership.

---

# Runtime Relationship

```text
[Runtime View — Execution Flow]

Theme Library
        │
        │
        ▼
Profile Content
└── Appearance State
        │
        ▼
Block Styling
        │
        ▼
Rendering
        │
        ▼
Public Profile
```

Explanation:

* Theme Library provides platform definitions.
* Appearance State stores the selected theme and user customization.
* Block Styling resolves the final visual values.
* Rendering generates the public profile.

Each component owns a distinct responsibility.

---

# Domain Boundaries

Appearance begins when Design changes visual configuration.

Appearance ends when Rendering receives the resolved Appearance State.

Anything after that belongs to Rendering.

Anything before that belongs to Profile or the Design editor.

Appearance never crosses these boundaries.

---

# External Dependencies

Appearance depends on:

```text
Profile Content

Theme Library
```

Appearance is consumed by:

```text
Rendering
```

Appearance has no dependency on:

```text
Account

Authentication

Social Accounts

Connected Accounts

Analytics

Out Links

QR Code

Settings

Public Profile
```

---

# Future Compatibility

The Appearance architecture intentionally supports future expansion without changing domain boundaries.

Examples include:

* accessibility preferences
* motion preferences
* premium theme collections
* seasonal themes
* additional theme families
* future visual capabilities

These additions extend Appearance without affecting:

* Profile Content ownership
* Rendering ownership
* Account ownership

---

# Architecture Canon

The following statements are permanent architectural rules.

## Canon 1

Appearance is part of the Profile domain (its design area).

---

## Canon 2

Appearance is responsible for visual configuration.

Rendering owns visual production.

---

## Canon 3

Appearance never owns profile content.

---

## Canon 4

Appearance never owns layout.

Layout belongs to the Rendering Domain.

Users cannot directly edit Layout in V1.

---

## Canon 5

Theme is a subsystem of Appearance.

Theme is not an independent domain.

---

## Canon 6

Theme Library is a platform-managed supporting resource.

Theme Definitions are never stored inside profile data.

Only the selected theme identifier and resolved visual configuration become part of the Appearance State.

---

## Canon 7

Appearance State is stored inside the Profile Content record.

Appearance does not own an independent persisted entity.

---

## Canon 8

Appearance is represented to users through the Design editor.

Users never interact with an "Appearance" editor.

---

## Canon 9

Appearance never introduces runtime concepts.

All runtime execution belongs to the Rendering Domain.

---

## Canon 10

Appearance is responsible for visual decisions.

Rendering owns visual implementation.

This responsibility boundary is permanent for Minime V1.

---

# Related Specifications

This document is the root specification of the Appearance system.

Theme subsystem:

```text
themes/

theme.catalog.specification.v1.md

theme.definition.specification.v1.md

theme.selection.specification.v1.md

theme.customization.specification.v1.md

theme.constraints.specification.v1.md
```

Future Appearance specifications:

```text
appearance.state.specification.v1.md

appearance.model.specification.v1.md
```

---

# References

This specification is governed by the approved Minime Architecture Canons.

* Product Architecture Canon
* Rendering Canon
* Theme Reorganization Canon
* Appearance Architecture Canon
* Domain Entity Canon
* Architecture View Canon

If any future specification conflicts with this document, the approved Architecture Canons take precedence.

---

# Summary

Appearance is the visual configuration domain of Minime V1.

It owns how Profile Content looks.

It never owns what Profile Content contains.

Appearance maintains one Appearance State per profile.

Appearance consumes the Theme Library while keeping Theme Definitions outside profile data.

Rendering consumes the Appearance State and remains solely responsible for producing the final Public Profile.

The separation between Profile Content, Appearance, and Rendering is a permanent architectural boundary of Minime V1.
